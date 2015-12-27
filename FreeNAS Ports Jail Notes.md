Created a jail in the FreeNAS interface using the Ports template
(called... "Ports". Particularly creative evening.)

```bash
# View jails  
Archivum# jls  
   JID  IP Address      Hostname                      Path  
     1  -               plexmediaserver_1             /mnt/archivum/jails/plexmediaserver_1  
     2  -               Ports                         /mnt/archivum/jails/Ports  
  
# Switch to jail  
Archivum# jexec 2 /bin/csh  
  
# Refresh ports tree  
Archivum# portsnap fetch && portsnap update  
  
# Get me some packages  
Archivum# pkg install bash vim-lite git  
  
# Change the shell  
Archivum# chsh -s /usr/local/bin/bash root
```

Edited `/etc/rc.conf` to set "`sshd_enable`" to YES.

In the FreeNAS interface, added a **Pre Init** command:

    export PATH=$PATH:/mnt/archivum/jails/Ports/usr/local/bin

This didn't seem to work...
