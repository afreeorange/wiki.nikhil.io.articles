Written for CentOS 6.4, Postfix 2.6.6.

-   Server is `nikhil.io`.
-   Mail users map to local accounts (i.e., in `/etc/passwd`).
-   DNS looks like this:

` [me@nikhil.io ~]$ dig nikhil.io -t MX +short`  
` 10 mail.nikhil.io.`

Overview
--------

Things were easier to set up after understanding these things, even
cursorily.

### Mail Transfer - Basic Idea

Lots of formal abbreviations! They are, luckily enough, [quite
sensible](http://dev.mutt.org/trac/wiki/MailConcept).

`   Sender > Server > Server > ... > Server > Receiver`  
`   (MUA)    (MTA)    (MTA)          (MTA)     (MUA)`

You can be a bit more granular:

`      Sender   >  Server > Server > ... >   Server     | Delivery |  >   Receiver`  
`   (MUA > MSA)    (MTA)    (MTA)          (MTA > MDA)  | complete |     (MRA > MUA)`

This separation of purpose is good since you can use [a variety of
applications and
topologies](http://twiki.org/cgi-bin/view/Wikilearn/EmailServerSketches)
at each stage. Lot of possibilities. E.g.

`   Sender > Postfix > Procmail > Clam Anti-Virus > SpamAssassin > Procmail > Fetchmail > Receiver`  
`   (MUA)    (MTA)     (                       MDA                        )     (MRA)       (MUA)`

### Open Relays

Where it is not required to (1) authenticate to your server, and/or (2)
be in the same network as the server to send email. This is very bad for
public-facing mail servers. From a simpler time when there were very few
email servers and everybody was nice to each other.

### Mailbox Formats

There are [quite a few](http://wiki2.dovecot.org/MailboxFormat), each
with its own pros and cons. I personally like
[Maildir](http://wiki2.dovecot.org/MailboxFormat/Maildir).

Installation
------------

`   yum install postfix`  
`   ln -s /usr/sbin/sendmail.postfix /etc/alternatives/mta --force`

Configuration
-------------

### The Basics

Postfix ships with sane and secure defaults. Here's stuff I changed in
`/etc/postfix/main.cf`

First set the hostname and domain

` myhostname = example.com`  
` mydomain = $myhostname`

Mail from this server will come from this domain.

` myorigin = $mydomain`

Accept mail on specified interface[^1] and all protocols (IPv4 and
IPv6[^2])

` inet_interfaces = all`  
` inet_protocols = all`

This server will think itself the final MTA (in the chain above) for
these domains:

` mydestination = $mydomain, localhost`

The server will only trust itself[^3]

` mynetworks_style = host`

Use the Maildir format for message delivery

` home_mailbox = Maildir/`

Change the banner for fun (and no version information)

` smtpd_banner = $myhostname ESMTP Why, hello there!`

### Testing

Restart the postfix service. Then from another computer,

`   orangebox:~ nikhil$ `**`telnet` `nikhil.io` `25`**  
`   Trying 96.126.123.32...`  
`   Connected to nikhil.io.`  
`   Escape character is '^]'.`  
`   `**`EHLO` `nikhil.io`**  
`   250-nikhil.io`  
`   250-PIPELINING`  
`   250-SIZE 10240000`  
`   250-VRFY`  
`   250-ETRN`  
`   250-ENHANCEDSTATUSCODES`  
`   250-8BITMIME`  
`   250 DSN`  
`   `**`MAIL` `FROM:` `testuser@internet.com`**  
`   250 2.1.0 Ok`  
`   `**`RCPT` `TO:` `me@nikhil.io`**  
`   250 2.1.5 Ok`  
`   `**`DATA`**  
`   354 End data with `<CR><LF>`.`<CR><LF>  
`   `**`Subject:` `Hello!`**  
`   `**`How` `have` `you` `been?`**  
`   `**`.`**  
`   250 2.0.0 Ok: queued as 7602C72C2`  
`   `**`QUIT`**

This should:

-   Work for a valid user

` MAIL FROM: postmaster@nikhil.io`  
` 250 2.1.0 Ok`

-   Not work an invalid user

` RCPT TO: nonexistent@nikhil.io`  
` 550 5.1.1 `<nonexistent@nikhil.io>`: Recipient address rejected: User unknown in local recipient table`

-   Deliver a mail to your home folder! The "`Maildir`" folder will be
    created automagically.

` ~/Maildir`  
`         ├── cur`  
`         ├── new`  
`         │   └── 1377029606.Vca00I4025fM640219.nikhil.io`  
`         └── tmp`

### Some restrictions

Stepping throught the telnet output, start adding some restrictions to
the client connection[^4]:

` smtpd_client_restrictions = permit_sasl_authenticated, reject_unknown_client_hostname`

Then the `HELO`[^5] command

` smtpd_helo_restrictions = reject_unknown_helo_hostname, reject_non_fqdn_helo_hostname, reject_invalid_helo_hostname`

`MAIL FROM`[^6]:

` smtpd_sender_login_maps = pcre:/etc/postfix/login_maps.pcre`  
` smtpd_sender_restrictions = reject_non_fqdn_sender, reject_sender_login_mismatch, reject_unknown_sender_domain`

And finally, `RCPT TO`[^7]

` smtpd_recipient_restrictions = reject_non_fqdn_recipient`

Add this to `/etc/postfix/login_maps.pcre`[^8]

` /^(.*)@nikhil.io$/   ${1}`

Test away! You should see good errors like:

` 450 4.1.8 `<askljdas@lksjdklfsdjf.com>`: Sender address rejected: Domain not found`  
` 450 4.7.1 `<blarghhh>`: Helo command rejected: Host not found`  
` 553 5.7.1 `<me@nikhil.io>`: Sender address rejected: not logged in`

Now add a way in which you an log in to the server to send messages.

### SASL Authentication

Will use [Cyrus](http://cyrusimap.web.cmu.edu/docs/cyrus-sasl/2.1.23/).
Postfix uses it [by
default](http://www.postfix.org/SASL_README.html#server_sasl_enable).
You can see what other libraries Postfix was compiled with support for
as well:

` [root@nikhil ~]# `**`postconf` `-a`**  
` cyrus`  
` dovecot`

You can then see what authentication methods Cyrus supports:

` [root@nikhil !]# `**`saslauthd` `-v`**  
` saslauthd 2.1.23`  
` authentication mechanisms: getpwent kerberos5 pam rimap shadow ldap`

Since we're dealing with local accounts, let's tell Cyrus to use
`/etc/shadow`. Open `/etc/sysconfig/saslauthd`:

` SOCKETDIR=/var/run/saslauthd`  
` `**`MECH=shadow`**  
` FLAGS=`

Start the service

` service saslauthd start`

Test!

` [root@nikhil !]# `**`testsaslauthd` `-u` `testuser` `-p`
`secretpassword`**  
` 0: OK "Success."`

Now tell Postfix to use Cyrus in `/etc/postfix/main.cf`

` smtpd_sasl_auth_enable = yes`  
` smtpd_sasl_type = cyrus`

Set [some security
options](http://www.postfix.org/SASL_README.html#smtpd_sasl_security_options)

` smtpd_sasl_security_options = noanonymous noplaintext`

Restart the Postfix service. Test via telnet

` [root@toolkit ~]# telnet nikhil.io 25`  
` Trying 96.126.123.32...`  
` Connected to nikhil.io.`  
` Escape character is '^]'.`  
` 220 nikhil.io ESMTP Hello. Please be nice.`  
` `**`AUTH` `PLAIN` `An8o0tjsHojfDausWtzblk4bnZA`**  
` 235 2.7.0 Authentication successful`

Generate the funky MD5 output with your username and password:

` echo -ne '\000user\000password' | base64`

Here's another problem: the authentication is done in the clear. 'They'
can see your username and password. Let's fix that:

Miscellaneous
-------------

### "warning: dict\_nis\_init:"

Disable NIS lookups

` alias_maps = hash:/etc/aliases`

### "Relay Access Denied"

[This explains
it](http://serverfault.com/questions/321864/postfix-relay-access-denied)
very, very clearly.

### Setting up Virtual Users

This is almost always a good thing.

<http://www.postfix.org/VIRTUAL_README.html>
<http://www.postfix.org/LOCAL_RECIPIENT_README.html>
<https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Server_Security-Securing_Postfix.html>

### `/etc/postfix/master.cf`

Made sure that [the `submission`
service](http://en.wikipedia.org/wiki/Mail_submission_agent) was *not*
enabled. Just plain `smtp`.

Great! Now we secure things.

Securing Postfix
----------------

1.  Only authenticated users can send mail (and as themselves)
2.  Encrypt all transactions using TLS

### Goal 1

You could use either Cyrus or Dovecot
[SASL](http://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer).
I'm sure many more SASL libraries exist, but these are pretty popular.
Once you pick one, Postfix will defer to it for authentication, and will
only send mail when the SASL layer says it's OK to do so. I'm picking
Dovecot.

Postfix should be compiled with support for your choice of library.
Check with:

`   [root@nikhil ~]# postconf -a`  
`   cyrus`  
`   dovecot`

Add this to `/etc/postfix/main.cf`

AUTH PLAIN AG1lAFAjdHQjbDFuZQ==

`   smtpd_sasl_auth_enable = yes`  
`   smtpd_sasl_local_domain = $mydomain`  
`   smtpd_sasl_path = private/auth`  
`   smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination`

### Goal 2

MIsc
----

smtpd\_client\_restrictions - Reject all client commands
smtpd\_sender\_restrictions - Reject MAIL FROM information
smtpd\_recipient\_restrictions - Reject RCPT TO information

Footnotes
---------

<references />
References
----------

-   [Email agent
    infrastructure](http://en.wikipedia.org/wiki/E-mail_agent_(infrastructure))
-   [How email
    works](http://en.kioskea.net/contents/116-how-email-works-mta-mda-mua)
-   [Securing Postfix
    (Red Hat)](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Server_Security-Securing_Postfix.html)
-   [Relay and Access
    Control](http://www.postfix.org/SMTPD_ACCESS_README.html)
-   [Postfix SASL Howto](http://www.postfix.org/SASL_README.html)
-   [Cyrus SASL for Systems
    Administrators](http://www.sendmail.org/~ca/email/cyrus/sysadmin.html)

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")

[^1]: Can specify IP address also:  
    `
      inet_interfaces=all<br />
      inet_interfaces=eth0,eth1<br />
      inet_interfaces=38.9.127.1,10.0.1.23<br />
      inet_interfaces=mail.tux.com<br />
    `

[^2]: Default is IPv4

[^3]: Can trust network classes or subnets and specific IP addresses

[^4]: <http://www.postfix.org/postconf.5.html#smtpd_client_restrictions>

[^5]: <http://www.postfix.org/postconf.5.html#smtpd_helo_restrictions>

[^6]: <http://www.postfix.org/postconf.5.html#smtpd_sender_restrictions>

[^7]: <http://www.postfix.org/postconf.5.html#smtpd_recipient_restrictions>

[^8]: <http://serverfault.com/a/318432>
