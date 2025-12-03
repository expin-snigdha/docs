### Set VNC Password
On dev machine , set your VNC password using `vncpasswd`.

### Create a VNC Session
Create or edit `vim /home/$USER/.vnc/xstartup` directory and add the following content:
```
#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4
```

```
tw-smc-4> chmod +x ~/.vnc/xstartup
```

Save the file, and then from your laptop, run the following command:
```
laptop> ssh tw-smc-4 vncserver -geometry 1600x1000 -depth 24 :<SCREEN_ID>
```
> **Alert:** 
   * If the `SCREEN_ID` is taken, then vnc will assign another - so note this down.
   * This is the recommended way of starting VNC so as not to have any environment dependencies on the GUI session.

###  Install VNC Viewer

To connect to VNC, download vnc viewer from `https://www.realvnc.com/en/connect/download/viewer` and follow onscreen instructions to install the software.

###  Connecting to VNC

Connect to the session using <hostname>:<SCREEN_ID> say, `tw-smc-4:11`. If all went well, you'll see Xfce4 desktop.


### Xfce4 recommended settings

Within VNC session, Xfce4 window manager is invoked. Go to settings and adjust the following:

1. Turn off screen saver and password lock

2. Use single color desktop background (preferably, Black color)

3. Since VNC sessions can be easily shared across users for debug, using a simple vnc password is recommended.

### VNC Viewer settings

Several default settings need to be changed for efficient use and development via VNC. Here are the details:

##### Expert settings

| **Parameter**                | **Value**                     |
|------------------------------|-------------------------------|
| `AcceptBell`                 | `False`                       |
| `EnableRemotePrinting`       | `False`                       |
| `LeftCmdKey`                 | `Super_L`                     |
| `LeftOptKey`                 | `Alt_L`                       |
| `RightCmdKey`                | `Super_R`                     |
| `RightOptKey`                | `Super_L`                     |
| `SendMediaKeys`              | `False`                       |
| `SendSpecialKeys`            | `False`                       |
| `SessionRecordAllowUserControl` | `False`                  |

