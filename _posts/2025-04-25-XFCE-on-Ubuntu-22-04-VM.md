---
layout: post
title:  "XFCE4 on Ubuntu 22.04 Virtual Machine"
date:   2025-04-25
tags: linux X11 
---

This post outlines the process of installing the XFCE desktop environment and configuring a VNC server for remote graphical access on an Ubuntu 22.04 server without a monitor. For a foundational understanding of the X Window System, refer to the earlier blog [Understanding and using the X Window System](https://srblog.github.io/2024/02/04/User-Guide-to-Linux-X11-Client-Server.html).

# XFCE Desktop Environment 

XFCE is a lightweight desktop environmnet ideal for low-resource computing infrastructure like lightweight VMs in the cloud. 
Similarly, `tigervnc` is a lightweight X-Server suitable for VMs to access the XFCE desktop remotely.

**Install the XFCE desktop environment and the VNC server**

{% highlight bash %}
$ sudo apt install xfce4 xfce4-goodies tigervnc-standalone-server
{% endhighlight %}

In most linux distros, the package is called `tigervnc-server`. After installing the server, a user needs to create a password.

{% highlight bash %}
$ vncpasswd
{% endhighlight %}

This will create a directory `~/.vnc` with all config, log and process files for the VNC server. If it does not exist, create a config file `~/.vnc/config` with some example parameters.

{% highlight bash %}
geometry=1024x768
localhost=no
{% endhighlight %}

The `geometry` options set the xfce desktop size on your remote desktop. In some default installation, the vncserver starts with the `localohost=yes` option that will not accept any connections from outside.

**Opening Ports in Firewall**

The TCP port utilized by the VNC server is decided by the server's starting display number. Without a specified display number, the server defaults to the lowest free one. VNC connections occur on port `5900+` display. Here, choosing display number `:1` means connecting to remote port `5901`. Typically, you might need to allow port access via the firewall. The commands will depend on the linux distro.

{% highlight bash %}
ufw allow 5901/tcp
{% endhighlight %}

**Setting up xstartup**

Create (if does not exist) and add the following shell comands to `~/.vnc/xstartup`


{% highlight bash %}
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
/usr/bin/startxfce4
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
x-window-manager &

{% endhighlight %}

Using `unset SESSION_MANAGER` prevents conflicts with other session managers when starting a desktop in a VNC session. Similarly, `unset DBUS_SESSION_BUS_ADDRESS` initiates a fresh D-Bus session for VNC, avoiding communication issues. `/usr/bin/startxfce4` then starts the XFCE desktop environment as the main interface for VNC connections.

`[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup`: Executes the script if /etc/vnc/xstartup is present and executable, allowing for session initialization. If executed after startxfce4, it indicates startxfce4 failed or ran in the background.

`[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources`
If the .Xresources file is readable, it loads X resources (e.g., terminal colors and fonts) to customize X applications in the VNC session.

`x-window-manager &` starts the window manager, which controls window placement, appearance, and behavior in an X session. This command is typically a symbolic link to the default system window manager such as `openbox`, `twm`, `ctwm`, or `mwm`.


Now you can start the server:

{% highlight bash %}
$ vncserver :1
{% endhighlight %}

Connect to the remote desktop using a VNC client, such as `tightVNC` for Windows. Enter the server's Public IP in the client's `Host Name` field to access the XFCE desktop environment remotely.

VNC connections are not encrypted by default. Use an SSH tunnel via PuTTY to secure them. Open PuTTY and enter `user@IP-Address` as the host name. In `Connections`, under `SSH > Tunnels`, set `5901` as source port and `IP-Address:5901` as destination, select `Local` and `Auto`. Click `Add`, save the session, and open the connection. Log in with your credentials. To access the XFCE session, use a VNC client like `tightVNC` and connect to `localhost:1`.

# Troubleshooting

Start the vncserver, and run `sudo netstat -plutn`. If it shows `127.0.0.1:1` or `::1`, it's running on localhost and inaccessible from outside. Ensure `~/.vnc/config` includes `localhost=no`. To check if the port is open on Windows, use `netstat -ano | grep 5901`, yielding output like:

{% highlight bash %}
TCP 127.0.0.1:5901 0.0.0.0:0 LISTENING 7856
TCP [::1]:5901 [::]:0 LISTENING 7856
{% endhighlight %}

# References 

- Rout S., "Understanding and using the X Window System", SRBlog 2024. [Link](https://srblog.github.io/2024/02/04/User-Guide-to-Linux-X11-Client-Server.html).

