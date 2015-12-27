Download and Install
--------------------

[Head over to
Sourceforge](http://sourceforge.net/project/showfiles.php?group_id=131204)
to download an RPM. Make sure that the major version of Python required
for the RPM matches that installed on your system.

    python --version

Now install the RPM:

    rpm -ivH DenyHosts-2.6-python2.4.noarch.rpm

Configuration
-------------

By default, the installation is placed in `/usr/share/denyhosts`. A few
steps are required before starting the daemon.

### denyhosts.cfg

This is the main config file. A sample file is provided and is called
`denyhosts.cfg-dist`. Make a copy of this file and start editing it:

    cp denyhosts.cfg-dist denyhosts.cfg  
    vim denyhosts.cfg

The file is beautifully self-explanatory, [as are the
FAQs](http://denyhosts.sourceforge.net/faq.html). Any further
explanation of the settings would be superfluous.

### The denyhosts daemon

Like the config file, copy the daemon:

    cp daemon-control-dist daemon-control  
    chown root daemon-control  
    chmod 700 daemon-control

No further configuration is necessary for the daemon if you're on RHEL.

### Starting the service

A symlink is necessary within `/etc/init.d`

    cd /etc/init.d/  
    ln -s /usr/share/denyhosts/daemon-control denyhosts  
    ./denyhosts start

### Run at startup

    chkconfig --add denyhosts  
    chkconfig denyhosts on

Other Notes
-----------

### Trusted IPs

The default `WORK_DIR` is `/usr/share/denyhosts/data`. An alternative is
`/var/lib/denyhosts`. An important consideration is the `allowed-hosts`
file, which lets you add an IP/range or domain as a 'trusted' source
which won't be banned (this can be fine-tuned in the `denyhosts.cfg`
file.) For example:

    19.67.35.*  
    jhu.edu

### Removing IPs

If ever you need to do this, you will have to remove them from these
files:

    /etc/hosts.deny  
    /var/lib/denyhosts/hosts  
    /var/lib/denyhosts/hosts-restricted  
    /var/lib/denyhosts/hosts-root  
    /var/lib/denyhosts/hosts-valid  
    /var/lib/denyhosts/users-hosts

It is a good idea to add trusted hosts to the `allowed-hosts` file after
this and restart the service. You can also use this script:

```bash
#!/bin/bash
 
# denyhosts-remove.sh
#
# AUTHOR: Tommy Butler, email: $ echo YWNlQHRvbW15YnV0bGVyLm1lCg==|base64 -d
# VERSION: 1.0
#
# SUMMARY:
# Use this script to Remove an IP address ban that has been errantly blacklisted
# by denyhosts - the ubiquitous and unforgiving brute-force attack protection
# service so often used on Linux boxen.
#
# INSTALL:
# Usage: Put this script somewhere in your $PATH, and execute it as root or
# with sudo.  Call it directly or with an IP address argument.  Multiple IP
# address arguments are not supported.  You'll need to `chmod +x` it first.
#
# LICENSE:
# GNU GPL 1.0
# Copyright 2011 Tommy Butler, All rights reserved
 
BASE_PATH="/var/lib/denyhosts";
IP=$1
 
if [[ "`/usr/bin/id -u`" != "0" ]]; then
   echo "Run this script as root or with sudo or app can't run correctly.  Aborted."
   exit 1;
fi
 
cd $BASE_PATH
 
if [[ "`pwd`" != "$BASE_PATH" ]]; then
   echo "Couldn't cd to $BASE_PATH.  Abort."
   exit 1;
fi
 
if [[ "$IP" == "" ]]; then
   echo "Enter the IP address you want to un-ban"
   read IP
fi
 
if [[ "$IP" == "" ]]; then
   echo "No IP address given.  Abort."
   exit 1;
fi
 
/etc/init.d/denyhosts stop
 
/usr/bin/perl -pi -e "s/^.*?$IP.*\n//g" /etc/hosts.deny *
 
/etc/init.d/denyhosts start
 
exit $?
```

### Using netfilter itself

While `denyhosts` is pretty good with regard to features, you can do a
basic 'bounce' with the IPT\_RECENT module.

    iptables -N SSH_CHECK  
    iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j SSH_CHECK  
    iptables -A SSH_CHECK -m state --state NEW -m recent --set --name SSH  
    iptables -A SSH_CHECK -m state --state NEW -m recent --update --seconds 60 --hitcount 4 --name SSH  
    iptables -A SSH_CHECK -m state --state NEW -m recent --rcheck --seconds 60 --hitcount 4 --name SSH -j DROP

### Test, test, test, test...

Enough said :)
