[TOC]

Pre-Flight
----------

-   The puppet server is `puppet.example.com` (CentOS 5.8 x64)
-   The puppet client is `client.example.com` (CentOS 6.2 x64)
-   Trying puppet 2.6+
-   The default ports are **8140** on server and **8139** on client; we
    won't be changing this.
-   See [Tweaking Puppet (and
    other notes)](Tweaking_Puppet_(and_other_notes) "wikilink") when
    you're finished here.

On Server
---------

    yum install puppet-server

Now edit `/etc/sysconfig/puppetmaster` and uncomment these two lines:

    PUPPETMASTER_MANIFEST=/etc/puppet/manifests/site.pp
    PUPPETMASTER_LOG=syslog

The site-wide manifest is `/etc/puppet/manifests/site.pp`. Let's add
this:

```ruby
# Define a few test classes
class testclass1 {
  file { "/tmp/test1":
    ensure => present,
    mode   => 644,
    owner  => root,
    group  => root
  }
}
class testclass2 {
  file { "/tmp/test2":
    ensure => present,
    mode   => 700,
    owner  => nobody,
    group  => nobody
  }
}

# Every node has this file
node default {
  include testclass1
}

# This particular node is a little different
node 'client.example.com' inherits default {
  include testclass2
}
```

Start the puppetmaster:

    service puppetmaster start

Configuring the Client
----------------------

    yum install puppet

Now edit `/etc/sysconfig/puppet` and uncomment these:

    PUPPET_SERVER=puppet.example.com  
    PUPPET_LOG=/var/log/puppet/puppet.log

Test your configuration by:

*   Running `/etc/init.d/puppet once --verbose`
*   Then tailing `/var/log/puppet/puppet.log`

        Thu Apr 12 13:52:12 -0500 2012 Puppet (notice): Reopening log files  
        Thu Apr 12 13:52:12 -0500 2012 Puppet (info): Creating a new SSL key for client.example.com  
        Thu Apr 12 13:52:12 -0500 2012 Puppet (info): Caching certificate for ca  
        Thu Apr 12 13:52:13 -0500 2012 Puppet (info): Creating a new SSL certificate request for client.example.com  
        Thu Apr 12 13:52:13 -0500 2012 Puppet (info): Certificate Request fingerprint (md5): A8:FE:8B:19:A8:9F:23:4C:19:27:65:7F:98:4D:E2:E6

You now need to sign the SSL request on the server. See the list of SSL
certificates:

    [root@sauron manifests]# puppetca --list -a
      client.example.com (A8:FE:8B:19:A8:9F:23:4C:19:27:65:7F:98:4D:E2:E6)  
    + puppet.example.com (7A:78:B2:B8:78:F3:26:53:23:1C:6B:5D:E0:40:C6:06) (alt names: [DNS:puppet](DNS:puppet), [DNS:puppet.example.com](DNS:puppet.example.com))

A "+" sign indicates a signed certificate. Sign the request:

    [root@sauron manifests]# puppetca --sign client.example.com
    notice: Signed certificate request for client.example.com  
    notice: Removing file Puppet::SSL::CertificateRequest client.example.com at '/var/lib/puppet/ssl/ca/requests/client.example.com.pem'

All should be well, so start the puppet service and make sure it starts
at boot (you could've done this with puppet!):

    service puppet start  
    chkconfig --level 345 puppet on

Setting up file services
------------------------

I edit `/etc/puppet/fileserver.conf` to add this:

    [files]  
     path /var/lib/puppet/files  
     allow 128.255.22.0/24

Note the **\[files\]** stub above. Created a sample file:

    cd /var/lib/puppet/files  
    mkdir -p etc/nikhil.conf  
    echo "Testing" > etc/nikhil.conf

Now in the manifest, add this:

    file { "/etc/nikhil.conf":  
        ensure => present,  
        owner => nobody,  
        group => root,  
        mode => 770  
        source => "puppet:///files/etc/nikhil.conf"  
    }

You can kick the puppet to see the file created with the contents on the
server.

**Note carefully** that

*   You'd have to specify the puppetmaster with two slashes (e.g.
    `puppet:*//*master.tld/files/nikhil.conf`)
*   You could omit the master using three slashes (e.g.
    `puppet:*///*files/etc/nikhil.conf`)

Issues
------

### Versioning

The puppet client version should be equal to or lower than the server
version. This one fact will save you a lot of trouble.

### Could not request certificate: Retrieved certificate does not match private key; please remove certificate from server and regenerate it with the current key

    # ON CLIENT: Remove all keys  
    rm -rfv /var/lib/puppet/ssl/*  
      
    # ON CLIENT: Make sure that the user puppet can write to /var/lib/puppet/ssl/  
      
    # ON SERVER: Revoke all certificates signed for client  
    puppetca --revoke client.example.com  
    puppetca --clean client.example.com

Then try again. Should work.

### Could not retrieve catalog from remote server: getaddrinfo: Name or service not known

*   Add an alias for `puppet` to `/etc/hosts` (DNS is better)
*   Add a `server = puppetmaster.domain.tld` to
    `/etc/puppet/puppet.conf` under the `[main]` section.

### Connection refused (2) when kicking puppet

Add a `listen = true` to the client's `/etc/puppet/puppet.conf` file.
See [the Puppet Configuration Reference](http://docs.puppetlabs.com/references/2.7.9/configuration.html)
for other directives.

### Could not retrieve catalog from remote server: Error 400 on SERVER: No support for http method POST

This happens if the client runs a later version than the server. For
example, 2.6 on the server and 2.7 on the client.

### Could not retrieve catalog from remote server: certificate verify failed

Make sure that the time on both server and client are in sync. Restart
the NTP service.

### puppet host "is already running"

[Issue with 2.6.18.274 kernels](http://projects.puppetlabs.com/issues/4948). 
Update to some 2.6.18.300+ kernel and reboot.

Tweaks et al
------------

### Autosign Certificate requests

Add this to the bottom of `/etc/puppet/puppet.conf` on the puppetmaster:

    [master]  
      autosign = true

You can also, apparently, create `/etc/puppet/autosign.conf` and append
the domain or CIDR for which the master will autosign.

    *.example.com  
    10.212.8.0/24

### 'Kicking' a Puppet

Clients, by default, listen on port 8139. Set up `/etc/puppet/auth.conf`
to have these lines:

    path /run  
    auth any  
    method save  
    allow puppet.server.com  
      
    # Must be above these lines!  
    path /  
    auth any

Restart the puppet daemon. Now kick it from the server:

    puppet kick --host client.domain.tld

An exit status of 0 is good. You *can* kick all clients, but will need a
LDAP.

### Debugging the Client

    service puppet stop  
    puppet agent --listen --debug --no-daemonize --verbose

### Check your manifest syntax

    puppet --parseonly manifest.pp

Sources
-------

*   http://archive09.linux.com/feature/143893
*   http://people.redhat.com/dlutter/puppet-app.html
*   http://projects.puppetlabs.com/projects/puppet/wiki/Core_Types_Cheat_Sheet
*   http://www.puppetcookbook.com/posts/creating-a-directory-tree.html
*   [Puppet Errors Explained](http://bitcube.co.uk/content/puppet-errors-explained)
*   [Tips and Tricks to debug puppet](http://www.devco.net/archives/2009/08/19/tips_and_tricks_for_puppet_debugging.php)
*   http://docs.puppetlabs.com/man/cert.html
