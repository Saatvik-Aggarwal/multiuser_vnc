# Notes on installing VNC for multi-user use
Following are notes on installing multi-user VNC on Kali 2018.3.

## Assumptions

* sssd already configured to allow logins
* oddjob-mkhomedir is also installed to automatically create users' home directories if they don't exist

## Disclaimers

* This configuration isn't the most secure.  When a user logs in, they are temporarily assigned a VNC port to use.  Anyone (knowing the password) can connect to that same port.  Since this configuration uses SSSD for authentication, I believe that this issue is negated but can't yet prove it.

## Steps

1) Install TigerVNC via:

```c
apt-get install tigervnc-standalone-server
```

2) Edit /etc/gdm3/daemon.conf and add the following two lines to the xdmcp section

```c
[xdmcp]
Enable=true
Port=177
```

3) Restart the display manger by running:

```c
systemctl restart gdm
```

4) Run and kill TigerVNC by running the below.  Such will create the original config files.

```c
tigervncserver
tigervncserver -kill :1
```
***Note:*** Above should not be needed for multi-user mode.

5) Create /etc/systemd/system/xvnc.socker with the following content:

```c
[Unit]
Description=XVNC Server

[Socket]
ListenStream=5900
Accept=yes

[Install]
WantedBy=sockets.target
```

6) Create /etc/systemd/system/xvnc@.service with the following content:

```c
[Unit]
Description=XVNC Per-Connection Daemon

[Service]
ExecStart=-/usr/bin/Xvnc -inetd -query localhost -geometry 1920x1080 -once -SecurityTypes=None
User=nobody
StandardInput=socket
StandardError=syslog
```

7) Enable and start the xvnc service by running:

```c
systemctl enable xvnc.socket
systemctl start xvnc.socket
```

## Troubleshooting

1) If a "Session declined: Maximum Number of Sessions Reached" error is returned, try adding the following line to the xdmcp section of /etc/gdm3/daemon.conf

```c
MaxSessions=2
```

2) If the login screen is flickering or crashing, try removing gnome-sell via:

```c
apt-get purge gnome-shell
```

***Note:*** this is untested but carried over from the source, just in case.

3) Just a point to note: if you're running vnc in a VM (depending on the number of users), you may need to add memory and processors to the VM, to handle the resource consumption.

## Sources

* https://wiki.archlinux.org/index.php/XDMCP (specifically the gdm section)
* https://wiki.archlinux.org/index.php/TigerVNC (specifically the multi-user section)

