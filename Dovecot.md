-   Written for CentOS 7.1, Dovecot 2.2.10.
-   Users are system users (in `/etc/aliases`)
-   Mailbox style is `Maildir` (in their home folders.)
-   Certificates are [Comodo PositiveSSLs from
    NameCheap](https://www.namecheap.com/security/ssl-certificates/domain-validation.aspx)

Pre-Flight
----------

Getting your mail is not something which can always be done via telnet
(insecure) or SSH (e.g. in the case of virtual accounts.)[^1]
[Dovecot](http://www.dovecot.org/) allows you to get your mail using the
POP3 and/or IMAP protocols.

### On SSL

-   The Dovecot instance will use POP3S and IMAPS in addition to POP3
    and IMAP. When TLS properly implemented/initiated with the latter
    pair, there's really no reason why the former would be required.
    Seems to be [some](http://wiki.dovecot.org/SSL)
    [debate](https://support.google.com/mail/answer/1074635?hl=en&uls=en)
    about this.
-   The Comodo certificates were chosen since they would [work with
    Gmail](http://www.tomsguide.com/us/Gmail-SSL-POP3-Certificate-Self-Signed,news-16468.html)
    and most other MUAs.

Installation
------------

`   yum install dovecot`  
`   systemctl enable dovecot`

Configuration
-------------

Turn off SSL (for now) in `/etc/dovecot/conf.d/10-ssl.conf`.

`   ssl = no`

### Initial Configuration

Edit `/etc/dovecot/dovecot.conf` and set the protocols you want to serve

`   protocols = imap pop3`

Listen on IPv4 and IPv6 interfaces

`   listen = *, ::`

Location for run time data

`   base_dir = /var/run/dovecot/`

Now, in `/etc/dovecot/conf.d/10-mail.conf`, tell Dovecot where to find
the messages

`   mail_location = maildir:~/Maildir`

Start the service and make sure it's running

`   [root@example ~]# systemctl start dovecot`  
`   [root@example ~]# netstat -tulpn | grep dovecot`  
`   tcp   0      0 0.0.0.0:110      0.0.0.0:*         LISTEN      7183/dovecot`  
`   tcp   0      0 0.0.0.0:143      0.0.0.0:*         LISTEN      7183/dovecot`  
`   tcp   0      0 :::110           :::*              LISTEN      7183/dovecot`  
`   tcp   0      0 :::143           :::*              LISTEN      7183/dovecot`

### Testing

You can now telnet to either ports 110
([POP3](http://www.anta.net/misc/telnet-troubleshooting/pop.shtml)) or
143
([IMAP](http://www.anta.net/misc/telnet-troubleshooting/imap.shtml)).
The syntaxes differ quite a bit.

Make sure firewall is poked :)

### Securing

Now we use TLS with the POP3 and IMAP ports. All authentication and
message transfer will be done only over a secure connection.

Edit `/etc/dovecot/conf.d/10-ssl.conf` to mandate SSL

`   ssl = required`

And configure the certificates and keys you will use

`   ssl_cert = </etc/pki/tls/certs/example.com.crt`  
`   ssl_key = </etc/pki/tls/private/example.com.key`  
`   ssl_ca = </etc/pki/CA/certs/ca-bundle.pem`

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

### Other Notes

-   When creating a CA bundle, `cat` the lowest intermediate all the way
    up to the root CA. I.e., [go in reverse
    order](https://support.comodo.com/index.php?/Knowledgebase/Article/View/643/0/how-do-i-make-my-own-bundle-file-from-crt-files).

<!-- -->

-   When connecting via OpenSSL, note how POP3 and IMAP servers respond
    (with the greeting "I am ready"):

`   # IMAP`  
`   * OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE AUTH=PLAIN] I am ready.`

`   # POP3`  
`   +OK I am ready.`

:   If using Gmail as an MUA, it expects a POP3 server/response.

Footnotes & References
----------------------

-   [Testing a Dovecot
    installation](http://wiki.dovecot.org/TestInstallation)

<references />
[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")

[^1]: I suppose you could [use
    OpenSSL](Postfix#Doing_things_securely "wikilink")... but who does
    that?
