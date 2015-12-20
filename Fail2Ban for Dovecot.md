Installation
------------

`   yum install fail2ban`  
`   chkconfig fail2ban on`

Configuration
-------------

Now add this to `/etc/fail2ban/jail.conf`. Change the `sender` email.

`   [dovecot]`  
`   enabled  = true`  
`   filter   = dovecot`  
`   action   = iptables-multiport[name=dovecot, port="pop3,pop3s,imap,imaps", protocol=tcp]`  
`              sendmail-whois[name=dovecot, dest=root, sender=fail2ban@example.com]`  
`   logpath  = /var/log/maillog`  
`   maxretry = 10`  
`   findtime = 1200`  
`   bantime  = -10`

See [the
manual](http://www.fail2ban.org/wiki/index.php/MANUAL_0_8#Jail_Options)
for an explanation of the options. In this configuration, anyone
attempting to authenticate unsuccessfully 10 times will be banned
permanently.

Fail2Ban will try looking for a configuration/filter file called
"`dovecot.conf`" in the filters directory, `/etc/fail2ban/filters.d`.
Create it and add this if it doesn't exist for some reason:

`   [Definition]`  
`   failregex = .*(?:pop3-login|imap-login):.*(?:Authentication failure|Aborted login \(auth failed|Aborted login \(tried to use disabled|Disconnected \(auth failed).*rip=(?P`<host>`\S*),.*`  
`   ignoreregex =`

Testing
-------

Fail2Ban comes with a handy-dandy regex testing tool.

`   fail2ban-regex /vaar/log/maillog /etc/fail2ban/filter.d/dovecot.conf`

Using the Service
-----------------

`   service fail2ban start`

Check its status

`   [root@example ~]# service fail2ban status`  
`   Fail2ban (pid 15919) is running...`  
`   Status`  
`   |- Number of jail:  1`  
``    `- Jail list:       dovecot ``

You should have gotten an email from the service with the subject
"**\[Fail2Ban\] dovecot: started**". Check `/var/log/messages` for
banned IPs

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
