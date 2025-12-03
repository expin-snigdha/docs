### Prerequisites

1. [VPN] (add page ink)
2. LDAP Credentials

### SSH to one of the [interactive] nodes in the colo

1. Once connected to the VPN service, SSH into any of the machines
```
laptop> ssh $USER@
```
2. Copy over initialization files (**one time only**)
```
> source /auto/tools/modulefiles/setup/setup.sh
```
3. Generate SSH Key (**one time only**)
```
tw-smc-4> ssh-keygen -t ed25519
```
4. Copy the public key for password-less access within colo nodes (**one time only**)
```
tw-smc-4> ssh-copy-id -i ~/.ssh/id_ed25519 $USER@tw-smc-4
```
Enter your LDAP password when prompted. This adds your colo public key to the ~/.ssh/authorized_keys.

5. Test password-less login from one node to another `ssh $USER@tw-smc-3`. You should be able to log in without entering a password.

### Enable password-less SSH access to colo machines from your laptop

1. Generate SSH Key on your laptop
```
laptop> ssh-keygen -t ed25519
```
2. Copy the public key from your laptop to tw-smc-4
```
laptop> ssh-copy-id -i ~/.ssh/id_ed25519 $USER@tw-smc-4
```
Enter your LDAP password when prompted. This adds your public key to the ~/.ssh/authorized_keys file on tw-smc-4.

3. Test password-less login from your laptop by executing `ssh $USER@tw-smc-4`. You should be able to log in without entering a password.

