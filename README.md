# Notes on installing VNC for multi-user use
Following are notes on installing multi-user VNC on Ubuntu 20.04


## Disclaimers

* This configuration isn't the most secure.  When a user logs in, they are temporarily assigned a VNC port to use.  Anyone (knowing the password) can connect to that same port. 

## Steps

1) Install TigerVNC via:

```c
apt-get install tigervnc-standalone-server
```

2) Edit /etc/gdm3/custom.conf and add the following four lines to the xdmcp section. Change "6" to however many you need. 

```c
[xdmcp]
Enable=true
Port=177
MaxSessions=6
DisplaysPerHost=6
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

5) Create /etc/systemd/system/xvnc.socket with the following content:

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

You should be done!

