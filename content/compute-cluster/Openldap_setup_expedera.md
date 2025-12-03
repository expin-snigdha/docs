# OpenLDAP Setup Guide for AlmaLinux 8.10 - Expedera

Complete working commands used to set up OpenLDAP for Expedera on AlmaLinux 8.10.
These are the exact commands that worked successfully.

**Server IP:** 172.21.0.111  
**Base DN:** dc=expedera,dc=com  
**Admin DN:** cn=admin,dc=expedera,dc=com

---

## 1. Installation

```bash
# Enable PowerTools repository
sudo dnf config-manager --set-enabled powertools

# Install packages
sudo dnf install -y openldap openldap-servers openldap-clients

# Start and enable service
sudo systemctl start slapd
sudo systemctl enable slapd
```

---

## 2. Basic Configuration

```bash
# Generate admin password hash
sudo slappasswd
# Enter password when prompted
# Example output: {SSHA}oxggcO9AgRaheJHekAeVpK+fpcz2Z/Zu
# SAVE THIS HASH!

# Check database type (should show mdb)
sudo ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config "(olcDatabase=*)" dn | grep olcDatabase

# Configure base DN and admin password (use YOUR hash from slappasswd above)
cat > ~/db.ldif << 'EOF'
dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=expedera,dc=com

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=expedera,dc=com

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}oxggcO9AgRaheJHekAeVpK+fpcz2Z/Zu
EOF

# NOTE: Replace {SSHA}oxggcO9AgRaheJHekAeVpK+fpcz2Z/Zu with YOUR actual hash

# Apply configuration
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ~/db.ldif

# Add required schemas
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

# Configure ACLs
cat > ~/acl.ldif << 'EOF'
dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to * by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth manage by * break
olcAccess: {1}to attrs=userPassword by self write by anonymous auth by * none
olcAccess: {2}to * by self write by dn.base="cn=admin,dc=expedera,dc=com" write by * read
EOF

sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ~/acl.ldif
```

---

## 3. Create Base Structure

```bash
# Create base organizational units
cat > ~/base.ldif << 'EOF'
dn: dc=expedera,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: Expedera
dc: expedera

dn: cn=admin,dc=expedera,dc=com
objectClass: organizationalRole
cn: admin
description: LDAP administrator

dn: ou=People,dc=expedera,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=expedera,dc=com
objectClass: organizationalUnit
ou: Groups
EOF

sudo ldapadd -Y EXTERNAL -H ldapi:/// -f ~/base.ldif

# Create default group
cat > ~/expedera_group.ldif << 'EOF'
dn: cn=expedera,ou=Groups,dc=expedera,dc=com
objectClass: posixGroup
cn: expedera
gidNumber: 5000
EOF

sudo ldapadd -Y EXTERNAL -H ldapi:/// -f ~/expedera_group.ldif
```

---

## 4. User Management Scripts

### Create ldap-adduser script

```bash
sudo cat > /usr/local/bin/ldap-adduser << 'SCRIPT'
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: ldap-adduser <username>"
    exit 1
fi

USERNAME=$1
NEXT_UID=$(ldapsearch -Y EXTERNAL -H ldapi:/// -b "ou=People,dc=expedera,dc=com" uidNumber 2>/dev/null | grep uidNumber | awk '{print $2}' | sort -n | tail -1)
if [ -z "$NEXT_UID" ]; then
    NEXT_UID=10000
else
    NEXT_UID=$((NEXT_UID + 1))
fi

echo "Creating user: $USERNAME"
echo "UID will be: $NEXT_UID"

while true; do
    echo -n "Enter surname (last name) [required]: "
    read SURNAME
    if [ -n "$SURNAME" ]; then
        break
    fi
    echo "Surname cannot be empty."
done

echo -n "Enter full name (or press Enter for $USERNAME): "
read FULLNAME
if [ -z "$FULLNAME" ]; then
    FULLNAME=$USERNAME
fi

echo "Generating password for $USERNAME..."
USER_PASSWORD=$(slappasswd)

cat > /tmp/${USERNAME}.ldif << EOF
dn: uid=${USERNAME},ou=People,dc=expedera,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: ${FULLNAME}
sn: ${SURNAME}
uid: ${USERNAME}
uidNumber: ${NEXT_UID}
gidNumber: 5000
homeDirectory: /home/${USERNAME}
loginShell: /bin/bash
userPassword: ${USER_PASSWORD}
mail: ${USERNAME}@expedera.in
description: User account
EOF

ldapadd -Y EXTERNAL -H ldapi:/// -f /tmp/${USERNAME}.ldif

if [ $? -eq 0 ]; then
    echo "User $USERNAME created successfully!"
    echo "UID: $NEXT_UID"
    rm /tmp/${USERNAME}.ldif
else
    echo "Failed to create user $USERNAME"
    echo "LDIF file saved at: /tmp/${USERNAME}.ldif"
fi
SCRIPT

sudo chmod +x /usr/local/bin/ldap-adduser
```

### Create ldap-listusers script

```bash
sudo cat > /usr/local/bin/ldap-listusers << 'SCRIPT'
#!/bin/bash
echo "LDAP Users in Expedera:"
echo "======================"
ldapsearch -Y EXTERNAL -H ldapi:/// -b "ou=People,dc=expedera,dc=com" "(objectClass=posixAccount)" uid uidNumber cn mail 2>/dev/null | grep -E "^(uid:|uidNumber:|cn:|mail:)" | sed 's/^/  /'
echo ""
SCRIPT

sudo chmod +x /usr/local/bin/ldap-listusers
```

### Create ldap-deluser script

```bash
sudo cat > /usr/local/bin/ldap-deluser << 'SCRIPT'
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: ldap-deluser <username>"
    exit 1
fi

USERNAME=$1

echo -n "Are you sure you want to delete user $USERNAME? (yes/no): "
read CONFIRM

if [ "$CONFIRM" != "yes" ]; then
    echo "User deletion cancelled."
    exit 0
fi

ldapdelete -Y EXTERNAL -H ldapi:/// "uid=${USERNAME},ou=People,dc=expedera,dc=com"

if [ $? -eq 0 ]; then
    echo "User $USERNAME deleted successfully!"
else
    echo "Failed to delete user $USERNAME"
fi
SCRIPT

sudo chmod +x /usr/local/bin/ldap-deluser
```

---

## 5. Verification Commands

```bash
# View entire LDAP tree
ldapsearch -x -b "dc=expedera,dc=com"

# List all users
sudo /usr/local/bin/ldap-listusers

# Search specific user
ldapsearch -x -b "dc=expedera,dc=com" "(uid=username)"

# View structure only
ldapsearch -x -b "dc=expedera,dc=com" -LLL dn
```

---

## 6. Usage Examples

```bash
# Add a new user
sudo /usr/local/bin/ldap-adduser john

# List all users
sudo /usr/local/bin/ldap-listusers

# Delete a user
sudo /usr/local/bin/ldap-deluser john
```

---

## 7. Firewall Configuration (Optional)

```bash
# If firewall is active, allow LDAP port
sudo firewall-cmd --permanent --add-service=ldap
sudo firewall-cmd --reload
```

---

## 8. Client Configuration (SSSD)

On client machines that need to authenticate against this LDAP server (IP: 172.21.0.111):

```bash
# Install packages
sudo dnf install -y sssd sssd-ldap oddjob-mkhomedir authselect

# Configure SSSD
sudo cat > /etc/sssd/sssd.conf << 'EOF'
[sssd]
services = nss, pam
config_file_version = 2
domains = expedera.com

[domain/expedera.com]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://172.21.0.111
ldap_search_base = dc=expedera,dc=com
ldap_id_use_start_tls = false
cache_credentials = true
ldap_tls_reqcert = allow
EOF

sudo chmod 600 /etc/sssd/sssd.conf

# Enable SSSD
sudo systemctl enable sssd
sudo systemctl restart sssd

# Enable auto home directory creation
sudo authselect select sssd with-mkhomedir --force
sudo systemctl enable oddjobd
sudo systemctl start oddjobd

# Test (replace 'snigdha' with actual username)
id snigdha
getent passwd snigdha
```

---

## 9. Useful Commands

```bash
# Restart LDAP service
sudo systemctl restart slapd

# Check service status
sudo systemctl status slapd

# View LDAP logs
sudo journalctl -u slapd -f

# Backup LDAP database
sudo slapcat -v -l /tmp/ldap-backup.ldif

# Restore LDAP database
sudo slapadd -v -l /tmp/ldap-backup.ldif
```

---

## Final Structure

```
dc=expedera,dc=com
├── cn=admin,dc=expedera,dc=com (administrator)
├── ou=People,dc=expedera,dc=com (users container)
│   ├── uid=user1 (UID: 10000)
│   └── uid=user2 (UID: 10001)
└── ou=Groups,dc=expedera,dc=com (groups container)
    └── cn=expedera (GID: 5000)
```
