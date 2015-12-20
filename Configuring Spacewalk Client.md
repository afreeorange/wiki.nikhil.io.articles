Installation Script
===================

I wrote a client installation script. See "[Spacewalk Setup
Script](Spacewalk_Setup_Script "wikilink")".

Pre-Flight
==========

-   The Spacewalk server is **spacewalk.example.com**
    -   CentOS 5.5 i386
-   The client to be registered is **client.example.com**
    -   CentOS 5.5 i386

Preparing the Client for Registration
=====================================

Install the necessary packages
------------------------------

On CentOS 5

` rpm -Uvh `[`http://spacewalk.redhat.com/yum/1.3/RHEL/5/i386/spacewalk-client-repo-1.3-1.el5.noarch.rpm`](http://spacewalk.redhat.com/yum/1.3/RHEL/5/i386/spacewalk-client-repo-1.3-1.el5.noarch.rpm)  
` yum install rhn-client-tools rhn-check rhn-setup rhnsd m2crypto yum-rhn-plugin`

Install the Spacewalk server certificate
----------------------------------------

You'll find the Spacewalk server certificate in the `/pub` folder:

` rpm -Uvh `[`http://spacewalk.example.com/pub/rhn-org-trusted-ssl-cert-1.0-1.noarch.rpm`](http://spacewalk.example.com/pub/rhn-org-trusted-ssl-cert-1.0-1.noarch.rpm)

This installs the certificate file (RHN-ORG-TRUSTED-SSL-CERT) in
`/usr/share/rhn`

Edit `/etc/sysconfig/rhn/up2date`
---------------------------------

I only changed the values of `serverURL` and `sslCACert`:

` sslCACert=/usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT`  
` serverURL=`[`https://spacewalk.example.com/XMLRPC`](https://spacewalk.example.com/XMLRPC)

Install EPEL
------------

You'll get GPG key signing errors (like "Public key for blah is not
installed") otherwise:

` rpm -Uvh `[`http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm`](http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm)

Register the client to the Spacewalk server
===========================================

Make sure you have two things:

1.  A properly configured `/etc/sysconfig/rhn/up2date` file
2.  The correct activation key

Then issue:

` rhnreg_ks --activationkey=1-d31a1b9465e576d5250de5da356b00a0`

Setting up Provisioning
=======================

Install these packages:

` yum -y install rhncfg rhncfg-actions rhncfg-client osad`

Enable the provisioning action on the client:

` rhn-actions-control --enable-all`

Read the `man` page for more fine-tuning. Now make sure that `osad` and
`rhnsd` are running:

` service osad start`  
` chkconfig --level 345 osad on`  
` service rhnsd start`  
` chkconfig --level 345 rhnsd on`

You should be all set now.

Setting up Monitoring
=====================

Install the `rhnmd` package and make sure it's running:

` yum -y install rhnmd`  
` service rhnmd start`  
` chkconfig --level 345 rhnmd on`

Make sure that the client firewall allows connections from port 4545
(default).

Installing this package creates a new user called `nocpulse`, whose
homedir is `/var/lib/nocpulse`. Now ask for the `nocpulse-identity`
public key from the server and put it in
`/var/lib/nocpulse/.ssh/authorized_keys`. Provided you've set up the
server properly, you should be all set now.

Errors and Miscellanea
----------------------

### "Local permission not set for action type configfiles.deploy" (code 42)

Make sure that you ran `rhn-actions-control`.

### 'Seeing' Server Commands

` osad -N -v -v -v -v`

Unregistering a Client
======================

Open `/etc/yum/pluginconf.d/rhnplugin.conf` and change

` enabled = 1`

to

` enabled = 0`

You may also want to clean up and reinstall RHN-related packages

` yum remove *rhn*`

Sources
=======

-   [Registering Clients - Spacewalk
    Wiki](https://fedorahosted.org/spacewalk/wiki/RegisteringClients)

[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
