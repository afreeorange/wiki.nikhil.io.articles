Written for CentOS 6.4, Postfix 2.6.6.

-   Server is `example.com`.
-   Mail users map to local accounts (i.e., in `/etc/passwd`).
-   DNS looks like this:

` [me@example.com ~]$ dig example.com -t MX +short`  
` 10 mail.example.com.`

-   Made sure that reverse DNS is set up properly.

Overview
--------

Things were easier to set up after understanding these things, even
cursorily[^1].

### Mail Transfer - Basic Idea

Lots of formal abbreviations! They are, luckily enough, [quite
sensible](http://dev.mutt.org/trac/wiki/MailConcept). Here's the basic
flow:

`   Sender > Server > Server > ... > Server > Receiver`  
`   (MUA)    (MTA)    (MTA)          (MTA)     (MUA)`

You can be a bit more granular:

`      Sender   >  Server > Server > ... >   Server     | Delivery |  >   Receiver`  
`   (MUA > MSA)    (MTA)    (MTA)          (MTA > MDA)  | complete |     (MRA > MUA)`

(MDAs, when local, are also called LDAs.)

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

` yum install postfix cyrus-sasl`  
` ln -s /usr/sbin/sendmail.postfix /etc/alternatives/mta --force`

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

Accept mail on specified interface[^2] and all protocols (IPv4 and
IPv6[^3])

` inet_interfaces = all`  
` inet_protocols = all`

This server will think itself the final MTA (in the chain above) for
these domains:

` mydestination = $mydomain, localhost`

The server will only trust itself[^4]

` mynetworks_style = host`

Use the Maildir format for message delivery

` home_mailbox = Maildir/`

Change the banner for fun (and no version information)

` smtpd_banner = $myhostname ESMTP Why, hello there!`

Now edit `[http://www.postfix.org/master.5.html /etc/postfix/master.cf]`
to enable the submission port[^5].

`   submission inet n       -       n       -       -       smtpd`

Uncomment any options ("-o"); we'll take care of these in later.

### Testing

Restart the postfix service. Then from another computer,

`   orangebox:~$ `**`telnet` `example.com` `25`**  
`   Trying 96.126.123.32...`  
`   Connected to example.com.`  
`   Escape character is '^]'.`  
`   `**`EHLO` `example.com`**  
`   250-example.com`  
`   250-PIPELINING`  
`   250-SIZE 10240000`  
`   250-VRFY`  
`   250-ETRN`  
`   250-ENHANCEDSTATUSCODES`  
`   250-8BITMIME`  
`   250 DSN`  
`   `**`MAIL` `FROM:` `testuser@internet.com`**  
`   250 2.1.0 Ok`  
`   `**`RCPT` `TO:` `me@example.com`**  
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

` MAIL FROM: postmaster@example.com`  
` 250 2.1.0 Ok`

-   Not work an invalid user

` RCPT TO: nonexistent@example.com`  
` 550 5.1.1 `<nonexistent@example.com>`: Recipient address rejected: User unknown in local recipient table`

-   Deliver a mail to your home folder! The "`Maildir`" folder will be
    created automagically.

` ~/Maildir`  
`         ├── cur`  
`         ├── new`  
`         │   └── 1377029606.Vca00I4025fM640219.example.com`  
`         └── tmp`

But this is done insecurely. Let's fix that.

### Doing things securely

Generate a self-signed certificate[^6]

` openssl req -new -x509 \`  
`             -newkey rsa:4096 \`  
`             -days 3650 \`  
`             -nodes \`  
`             -out /etc/pki/tls/certs/dovecot.crt \`  
`             -keyout /etc/pki/tls/private/dovecot.key`

` chmod o= /etc/pki/tls/private/dovecot.pem`

Now configure Postfix to use these certificates for TLS

` smtp_tls_security_level = may`  
` smtpd_tls_security_level = may`  
` smtpd_tls_cert_file = /etc/pki/tls/certs/postfix.crt`  
` smtpd_tls_key_file = /etc/pki/tls/private/postfix.key`  
` smtpd_tls_auth_only = yes`  
` smtpd_tls_loglevel = 3`  
` smtpd_tls_received_header = yes`  
` smtpd_tls_session_cache_timeout = 3600s`  
` tls_random_source = dev:/dev/urandom`

Restart Postfix. As always, see `/var/log/maillog` for any errors.

Now test. **Important** You have to use the OpenSSL client instead of
telnet from this point on! Watch out for non-zero "Verify return codes".
Avoid these by downloading the generated "`postfix.crt`" using it with
your OpenSSL command.

` openssl s_client -starttls smtp \`  
`                  -CAfile /path/to/postfix.crt \`  
`                  -connect example.com:25`

Another **important** warning is to *keep the former telnet commands
lowercase*. Else, the client will renegotiate every time you type
`RCPT TO`. [OpenSSL can waste your time like
that](http://archives.neohapsis.com/archives/postfix/2007-01/1334.html)!

### Some restrictions

Stepping throught the telnet output in the previous section, start
adding some restrictions to the client connection[^7]:

` smtpd_client_restrictions = reject_unknown_client_hostname, permit`

Then the
`[http://www.postfix.org/postconf.5.html#smtpd_helo_restrictions HELO]`
command

` smtpd_helo_required = yes`  
` smtpd_helo_restrictions = reject_unknown_helo_hostname, reject_non_fqdn_helo_hostname, reject_invalid_helo_hostname, permit`

`[http://www.postfix.org/postconf.5.html#smtpd_sender_restrictions MAIL FROM]`

` smtpd_sender_login_maps = pcre:/etc/postfix/login_maps.pcre`  
` smtpd_sender_restrictions = reject_non_fqdn_sender, reject_sender_login_mismatch, reject_unknown_sender_domain, permit`

And finally,
`[http://www.postfix.org/postconf.5.html#smtpd_recipient_restrictions RCPT TO]`.
This will allow you to relay messages (i.e. send email to *other*
domains) if you're SASL-authenticated.

` smtpd_recipient_restrictions = permit_sasl_authenticated, reject_unauth_destination, permit`

Add this to `/etc/postfix/login_maps.pcre`[^8].

` /^(.*)@example.com$/   ${1}`

Test away! You should see good errors like:

` 450 4.1.8 `<askljdas@lksjdklfsdjf.com>`: Sender address rejected: Domain not found`  
` 450 4.7.1 `<blarghhh>`: Helo command rejected: Host not found`  
` 553 5.7.1 `<me@example.com>`: Sender address rejected: not logged in`

Now add a way in which you an log in to the server remotely to send
messages through it.

### SASL Authentication

Will use [Cyrus](http://cyrusimap.web.cmu.edu/docs/cyrus-sasl/2.1.23/).
Postfix [uses it by
default](http://www.postfix.org/SASL_README.html#server_sasl_enable).
You can see what other libraries Postfix was compiled with support for
as well:

` [root@example ~]# `**`postconf` `-a`**  
` cyrus`  
` dovecot`

Install Cyrus

` yum install cyrus-sasl`

You can then see what authentication methods Cyrus supports:

` [root@example !]# `**`saslauthd` `-v`**  
` saslauthd 2.1.23`  
` authentication mechanisms: getpwent kerberos5 pam rimap shadow ldap`

Install the appropriate package. Since I'm using plain auth,

` yum install cyrus-sasl-plain`

Since we're dealing with local accounts, let's tell Cyrus to use
`/etc/shadow`. Open `/etc/sysconfig/saslauthd`:

` SOCKETDIR=/var/run/saslauthd`  
` `**`MECH=shadow`**  
` FLAGS=`

Start the service

` service saslauthd start`

Make sure it starts when you reboot your server

` chkconfig saslauthd on`

Test!

` [root@example !]# `**`testsaslauthd` `-u` `testuser` `-p`
`secretpassword`**  
` 0: OK "Success."`

Now tell Postfix to use Cyrus in `/etc/postfix/main.cf`

` smtpd_sasl_auth_enable = yes`  
` smtpd_sasl_type = cyrus`

Set [some security
options](http://www.postfix.org/SASL_README.html#smtpd_sasl_security_options)

` smtpd_sasl_security_options = noanonymous`

Restart the Postfix service. Test:

` [root@toolkit ~]# openssl s_client -starttls smtp -CAfile /path/to/postfix.crt -connect example.com:25`  
` (Certificate, connection info)`  
` ---`  
` 250 DSN`  
` `**`helo` `example.com`**  
` 250 example.com`  
` `**`auth` `plain` `An8o0tjsHojfDausWtzblk4bnZA`**  
` 235 2.7.0 Authentication successful`

Generate the funky MD5 output with your username and password:

` echo -ne '\000user\000password' | base64`

Preventing Spam, Bad Email, and DOS Attacks
-------------------------------------------

### Using Blocklists

Change `smtpd_recipient_restrictions` to add some blocklists[^9] and
other stringent policies:

` smtpd_recipient_restrictions = `  
`   permit_sasl_authenticated, `  
`   reject_invalid_hostname, `  
`   reject_non_fqdn_sender,`  
`   reject_non_fqdn_recipient,`  
`   reject_unauth_destination,`  
`   reject_rbl_client zen.spamhaus.org,`  
`   reject_rbl_client psbl.surriel.com,`  
`   reject_rbl_client bl.spamcop.net,`  
`   permit`

Most residential IPs are banned by blocklists, so keep that in mind when
testing your setup:

` 554 5.7.1 Service unavailable; Client host [173.29.77.33] blocked using zen.spamhaus.org; `  
` `[`http://www.spamhaus.org/query/bl?ip=173.29.77.33`](http://www.spamhaus.org/query/bl?ip=173.29.77.33)

### Using SPF

[Sender Policy Framework](http://www.openspf.org/Project_Overview)
prevents fake sender addresses from your domain. It's a great idea and
is something everyone should do[^10].

To empower Postfix with SPF, first install some required packages from
EPEL:

` yum install perl-core perl-Mail-SPF --enablerepo=epel`

I'm going to try [the Perl implementation of
SPF](https://launchpad.net/postfix-policyd-spf-perl/). [^11] Download,
extract, move to a good place:

` tar -xvzf postfix-policyd-spf-perl-2.010.tar.gz`  
` cd postfix-policyd-spf-perl-2.010`  
` mv postfix-policyd-spf-perl /usr/local/bin/`

Now set up `/etc/postfix/main.cf`. Add to `smtpd_recipient_restrictions`

` # Other options not shown for brevity`  
` smtpd_recipient_restrictions = `  
`   check_policy_service unix:private/policy-spf,`  
`   policy_time_limit = 3600 # Default is 1000; too short`[^12]

Then add to `/etc/postfix/master.cf`

` policy-spf unix  -       n       n       -       0       spawn`  
`     user=nobody  argv=/usr/bin/perl /usr/local/bin/postfix-policyd-spf-perl`

Restart the service. Send yourself an email from another service Gmail,
and look for SPF output in `/var/log/maillog`

### Some Limits on Interaction

The first setting is the remainder for the "limit"s

`   # Connection limits`  
`   anvil_rate_time_unit = 120s`  
`   smtpd_client_connection_rate_limit = 2400`  
`   smtpd_client_message_rate_limit = 12000`  
`   smtpd_error_sleep_time = 60`

### Prevent Abuse

[Greylisting](http://en.wikipedia.org/wiki/Greylisting) is a great
approach to fighting spam. The idea is that spammy mail servers do not
respect the RFC spec that, if an email couldn't be delivered initially,
they are to re-attempt delivery later.
[Postgrey](http://postgrey.schweikert.ch/) works well for this. By
default, it asks MTAs to attempt redelivery in 5 minutes.

`   yum install postgrey`  
`   service postgrey start`  
`   chkconfig postgrey on`

This will run on a Unix socket. The next step is to get Postfix to use
it. Edit `/etc/postfix/main.cf`.

`   # Other options not shown for brevity`  
`   smtpd_recipient_restrictions =`  
`       check_policy_service unix:postgrey/socket`

I lowered the default wait time to a minute by creating
`/etc/sysconfig/postgrey` and adding this:

`   OPTIONS=$OPTIONS" --delay=60"`

Restart Postfix and Postgrey. You'll see something like this in
`maillog` to make sure it's working:

`   postgrey[12582]: action=pass, reason=client whitelist, client_name=mail-qc0-f169.google.com, `  
`       client_address=209.85.216.169, sender=anand.nikhil@gmail.com, recipient=test@example.com`

Miscellaneous
-------------

### "warning: dict\_nis\_init:"

Disable NIS lookups

` alias_maps = hash:/etc/aliases`

### "Relay Access Denied"

Usually
[something](http://serverfault.com/questions/321864/postfix-relay-access-denied)
quite
[simple](http://serverfault.com/questions/192354/postfix-sasl-relay-access-denied-when-sending-from-outside-the-network).

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
-   [Postfix + SPF +
    CentOS6](http://www.thenoccave.com/2013/05/08/centos-6-postfix-spf-checking/)
-   [Postfix FAQ](http://www.seaglass.com/postfix/faq.html#chbnc)

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")

[^1]: [The Linode page](https://library.linode.com/mailserver) on mail
    servers is also a great overview

[^2]: Can specify IP address also:  
    `
      inet_interfaces=all<br />
      inet_interfaces=eth0,eth1<br />
      inet_interfaces=38.9.127.1,10.0.1.23<br />
      inet_interfaces=mail.tux.com<br />
    `

[^3]: Default is IPv4

[^4]: Can trust network classes or subnets and specific IP addresses

[^5]: I was a little confused about this but think I understand. Port 25
    is the standard SMTP port that used for MTA-to-MTA communication. So
    if you have a user who is behind an ISP connection that blocks port
    25 (for spam or other reasons like bad proxying), they can still
    send/submit mail to your server, even if it's not the final
    destination on the message envelope, on port 587.

[^6]: Can also use [StartSSL](http://www.startssl.com/) or
    [CACert](http://www.cacert.org/).

[^7]: <http://www.postfix.org/postconf.5.html#smtpd_client_restrictions>

[^8]: From [ServerFault](http://serverfault.com/a/318432). Postfix can
    use a *lot* more formats for controlled envelopes. See the output of
    `postconf -m`. For instance, I initally used this file (Specified
    with `hash:/path/to/file`):  
    `
    <nowiki>#</nowiki> Envelope sender Owner<br />
    me@example.com    me<br />
    `

[^9]: Of which there are [a lot
    available](http://www.dnsbl.info/dnsbl-list.php)

[^10]: To get started, [read about the
    syntax](http://www.openspf.org/SPF_Record_Syntax) or use [a
    wizard](http://spfwizard.com), then the [validation
    tool](http://www.kitterman.com/spf/validate.html).

[^11]: Tried to make the Python version work but ran into issues with
    Python3 and the `ipaddr` module.

[^12]: [`http://www.postfix.org/SMTPD_POLICY_README.html#client_config`](http://www.postfix.org/SMTPD_POLICY_README.html#client_config)
