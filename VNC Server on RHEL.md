Based on a 64-bit CentOS 5.8 box.

Install necessary packages
--------------------------

` yum -y install vnc vnc-server`

Set up VNC users
----------------

` useradd vncuser`  
` su vncuser`  
` vncpasswd`

Enter the password you'll use to connect. This creates a `.vnc` file in
the user's homedir. Now edit `~/.vnc/xstartup` to uncomment the lines
pertaining to a normal desktop:

` unset SESSION_MANAGER`  
` exec /etc/X11/xinit/xinitrc`

Set up the VNC configuration
----------------------------

I added this to `/etc/sysconfig/vncservers`:

` # No SSH tunneling`  
` VNCSERVERS="2:support"`  
` VNCSERVERARGS[2]="-geometry 800x600 -nolisten tcp -nohttpd"`

Set firewall rules
------------------

Look at the `2:support` above. The number is added to ports 5800, 5900
and 6000 for connections.

| Port | Function |---- | 5800+n | For Java-based VNC viewers (e.g. through a webstart application) |---- | 5900+n | VNC Client port |---- | 6000+n | X Server port |
|------|----------------|--------|------------------------------------------------------------------------|--------|-----------------------|--------|---------------|

At a bare minimum, port 590**2** must be open. If you want other fancy
stuff, open ports 580**2**, 590**2** and 600**2** (do this securely; see
section below).

Start the VNC Service
---------------------

` service vncserver start`

Test, test, test!

Using VNC Securely
------------------

To tunnel your VNC connection through SSH, add `-localhost` to
VNCSERVERARGS in `/etc/sysconfig/vncservers`. In the example above,

` VNCSERVERS="2:support"`  
` VNCSERVERARGS[2]="-geometry 800x600 -nolisten tcp -nohttpd `**`-localhost`**`"`

Restart the VNC service. We're now listening on port 5902 for *local
connections to that port only*.

### Client-side connection

Easy peasy:

` ssh -L 5902:localhost:5902 support@server.example.com -N`

Tunnels all requests on port 5902 on your computer to port 5902 on the
server ("-L") and doesn't execute any commands ("-N", port-forwarding
only.) You can add "-f" to push this into the background.

Troubleshooting
---------------

If you cannot start the VNC service (i.e. get a "FAILED"), make sure
that you do these in order:

1.  `useradd vncuser`
2.  `su vncuser`
3.  `vncpasswd vncuser`
4.  `exit`
5.  `service vncserver restart`

Step 2 is important! You need to *be* the user when setting your VNC
password. `vncpasswd vncuser` as root won't work.

Sources
-------

-   <http://wiki.centos.org/HowTos/VNC-Server>

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
