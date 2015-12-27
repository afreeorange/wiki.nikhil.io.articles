Installation
------------

    yum install fail2ban  
    chkconfig fail2ban on

Configuration
-------------

Now add this to `/etc/fail2ban/jail.conf`. Change the `sender` email.

    [dovecot]  
    enabled  = true  
    filter   = dovecot  
    action   = iptables-multiport[name=dovecot, port="pop3,pop3s,imap,imaps", protocol=tcp]  
               sendmail-whois[name=dovecot, dest=root, sender=fail2ban@example.com]  
    logpath  = /var/log/maillog  
    maxretry = 10  
    findtime = 1200  
    bantime  = -10

See [the manual](http://www.fail2ban.org/wiki/index.php/MANUAL_0_8#Jail_Options)
for an explanation of the options. In this configuration, anyone
attempting to authenticate unsuccessfully 10 times will be banned
permanently.

Fail2Ban will try looking for a configuration/filter file called
"`dovecot.conf`" in the filters directory, `/etc/fail2ban/filters.d`.
Create it and add this if it doesn't exist for some reason:

    [Definition]  
    failregex = .*(?:pop3-login|imap-login):.*(?:Authentication failure|Aborted login \(auth failed|Aborted login \(tried to use disabled|Disconnected \(auth failed).*rip=(?P<host>\S*),.*  
    ignoreregex =

Testing
-------

Fail2Ban comes with a handy-dandy regex testing tool.

    fail2ban-regex /vaar/log/maillog /etc/fail2ban/filter.d/dovecot.conf

You should issue `iptables -L` to verify that there's a Fail2Ban chain.
Now `telnet` to the POP3 port from another machine and try logging in
with some junk.

    Trying 198.81.129.107...  
    Connected to example.com (198.81.129.107).  
    Escape character is '^]'.  
    +OK Hello. Please be nice.  
    user hahahaha**  
    -ERR Plaintext authentication disallowed on non-secure (SSL/TLS) connections.

You'll see a bunch of these in `/var/log/maillog`:

    Sep 17 15:05:14 user dovecot: pop3-login: Aborted login (tried to use disabled plaintext auth): rip=72.21.81.85, lip=198.81.129.107

When you see 10 of them, wait a bit. You'll see an email from Fail2Ban
informing you that it's blocked an IP. To verify, issue
`iptables -L -n`. You'll see this somewhere:

    Chain fail2ban-dovecot (1 references)  
    target     prot opt source               destination  
    DROP       all  --  72.21.81.85        0.0.0.0/0

Nice. To unban, just remove the rule from the chain:

    iptables -D fail2ban-dovecot -s 72.21.81.85 -j DROP

Using the Service
-----------------

    service fail2ban start

Check its status

    [root@example ~]# service fail2ban status  
    Fail2ban (pid 15919) is running...  
    Status  
    |- Number of jail:  1  
    `- Jail list:       dovecot 

You should have gotten an email from the service with the subject
"**\[Fail2Ban\] dovecot: started**". Check `/var/log/messages` for
banned IPs



