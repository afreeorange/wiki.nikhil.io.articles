Written for CentOS 6.4, Dovecot 2.0.9. Users are system users, mailbox
style is Maildir (in their home accounts.)

Pre-Flight
----------

Getting your mail is not something which can always be done via telnet
(insecure) or SSH (e.g. in the case of virtual accounts.)[^1]
[Dovecot](http://www.dovecot.org/) allows you to get your mail using the
POP3 and/or IMAP protocols. It's fast and secure out of the box.

### On SSL

The Dovecot instance will use POP3S and IMAPS in addition to POP3 and
IMAP. When TLS properly implemented with the latter pair, there's really
no reason why the former would be required. Seems to be
[some](http://wiki.dovecot.org/SSL)
[confusion](https://support.google.com/mail/answer/1074635?hl=en&uls=en).

Installation
------------

`   yum install dovecot`  
`   chkconfig dovecot on`  
`   service dovecot start`

Configuration
-------------

Turn off SSL (for now) in `/etc/dovecot/10-ssl.conf`.

`   ssl = no`

### Initial Configuration

Edit `/etc/dovecot/dovecot.conf` and set the protocols you want to serve

`   protocols = imap pop3`

Listen on IPv4 and IPv6 interfaces

`   listen = *, ::`

Location for run time data

`   base_dir = /var/run/dovecot/`

Now, in `/etc/dovecot/10-mail.conf`, tell Dovecot where to find the
messages

`   mail_location = maildir:~/Maildir`

Start the service and make sure it's running

`   [root@example ~]# service dovecot start`  
`   [root@example ~]# netstat -tulpn | grep dovecot`  
`   tcp   0      0 0.0.0.0:110      0.0.0.0:*         LISTEN      7183/dovecot`  
`   tcp   0      0 0.0.0.0:143      0.0.0.0:*         LISTEN      7183/dovecot`  
`   tcp   0      0 :::110           :::*              LISTEN      7183/dovecot`  
`   tcp   0      0 :::143           :::*              LISTEN      7183/dovecot`

### Testing

You can now telnet to either ports 110
([POP3](http://techhelp.santovec.us/pop3telnet.htm)) or 143
([IMAP](http://www.anta.net/misc/telnet-troubleshooting/imap.shtml)).
The syntaxes differ quite a bit.

Make sure firewall is poked :)

### Securing

Now we use TLS with the POP3 and IMAP ports. All authentication and
message transfer will be done only after STARTTLS.

Edit `/etc/dovecot/10-ssl.conf` to enable SSL

`   ssl = yes`

And configure the certificates and keys you will use (`ssl_cert` and
`ssl_key`). If, like me, you're using self-signed certificates from
StartSSL, you'll need to specify the CA bundle as well (`ssl_ca`).

Now disable plaintext authentication in `/etc/dovecot/10-auth.conf`

`   disable_plaintext_auth = yes`

Restart the dovecot service. You'll see ports 993 and 995 in the
`netstat` output. Use OpenSSL to test the POP3S service first:

`   openssl s_client -connect example.com:995`

You should be able to log in and check some test messages. The IMAP
service should work fine as well.

Importantly, you should not be able to authenticate insecurely.

`   [root@example ~]# `**`telnet` `example.com` `110`**  
`   Trying 96.126.123.32...`  
`   Connected to example.com.`  
`   Escape character is '^]'.`  
`   +OK Dovecot ready.`  
`   `**`user` `testuser`**  
`   -ERR Plaintext authentication disallowed on non-secure (SSL/TLS) connections.`

This is good. Test like crazy!

References
----------

-   [Testing a Dovecot
    installation](http://wiki.dovecot.org/TestInstallation)

<references />
[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")

[^1]: I suppose you could [use
    OpenSSL](Postfix#Doing_things_securely "wikilink")... but who does
    that?
