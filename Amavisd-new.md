For CentOS 6.4, with Amavisd-new 2.8. Assuming you have
[ClamAV](ClamAV "wikilink") and [SpamAssassin](SpamAssassin "wikilink")
installed already.

[Amavisd-new](http://www.ijs.si/software/amavisd/) takes a message from
[Postfix](Postfix "wikilink"), gives it to content checkers like
[ClamAV](http://www.clamav.net/lang/en/) and
[SpamAssassin](http://spamassassin.apache.org/), and hands the message
back to Postfix, which then decides what to do with it (i.e., reject,
keep it in hold, and so on)[^1].

I learned a *lot* about this from [this excellent
guide](http://www.ijs.si/software/amavisd/README.postfix.htm).

Installation
------------

`   # Install`  
`   yum install amavisd-new clamav spamassassin`  
`   `  
`   # Start`  
`   chkconfig amavisd on; service amavisd start`  
`     `  
`   # Update`  
`   freshclam`  
`   sa-update`

You'll have to update only once; both ClamAV and SpamAssassin come with
their own `cron` jobs. Handy.

Setting up the Transport
------------------------

Unless you changed the defaults, the `amavisd` daemon will run on
localhost, on port 10024. Configuration is a two-step process.

### Transport Messages from Postfix to Amavis

You can ask Postfix to filter a message through whatever you want after
it is queued but before it is delivered to a mailbox. The filter can be
a defined as a pipe, a unix socket, or a TCP/IP socket.

We have the Amavis daemon listening on 127.0.0.1:10024. Let's tell
Postfix to filter its messages through that TCP/IP socket. In
`/etc/postfix/main.cf`, add the following:

`   content_filter = amavisd:[127.0.0.1]:10024`

This is of the form *transport:destination*. The first part should
correspond to a definition in `/etc/postfix/master.cf`. Let's add it:

`   amavisd unix    -       -       n       -       2       smtp`  
`       -o smtp_data_done_timeout=1200`  
`       -o smtp_send_xforward_command=yes`  
`       -o disable_dns_lookups=yes`  
`       -o max_use=20`

### From Amavis back to Postfix

`/etc/amavisd.conf` contains two options, `notify_method` and
`forward_method`. These are the destinations where Amavis will send
notifications and/or messages after processing. The default is an SMTP
host, listening at 127.0.0.1:10025. We can ask Postfix to listen at that
port, thereby letting it get back the messages it sent to Amavis.

This is again the form *transport:destination*, and must be defined in
`/etc/postfix/master.cf`.

`   127.0.0.1:10025 inet n  -       n       -       -       smtpd`  
`     -o content_filter=`  
`     -o local_recipient_maps=`  
`     -o relay_recipient_maps=`  
`     -o smtpd_restriction_classes=`  
`     -o smtpd_delay_reject=no`  
`     -o smtpd_client_restrictions=permit_mynetworks,reject`  
`     -o smtpd_helo_restrictions=`  
`     -o smtpd_sender_restrictions=`  
`     -o smtpd_recipient_restrictions=permit_mynetworks,reject`  
`     -o smtpd_data_restrictions=reject_unauth_pipelining`  
`     -o smtpd_end_of_data_restrictions=`  
`     -o mynetworks=127.0.0.0/8`  
`     -o smtpd_error_sleep_time=0`  
`     -o smtpd_soft_error_limit=1001`  
`     -o smtpd_hard_error_limit=1000`  
`     -o smtpd_client_connection_count_limit=0`  
`     -o smtpd_client_connection_rate_limit=0`  
`     -o receive_override_options=no_header_body_checks,no_unknown_recipient_checks`

Since the usual SMTP server checks were already applied by Postfix, we
set up an innocent/dumb/minimal SMTP 'server'.

Setting up Amavis
-----------------

Set the domain and hostnames

`   $mydomain = 'example.com';`  
`   $myhostname = 'host.example.com';`

Uncomment the notify and forward methods

`   $notify_method  = 'smtp:[127.0.0.1]:10025';`  
`   $forward_method = 'smtp:[127.0.0.1]:10025';`

Uncomment these lines from `/etc/amavisd.conf`

`   ['ClamAV-clamd',`  
`     \&ask_daemon, ["CONTSCAN {}\n", "/var/run/clamav/clamd.sock"],`  
`     qr/\bOK$/m, qr/\bFOUND$/m,`  
`     qr/^.*?: (?!Infected Archive)(.*) FOUND$/m ]`

Miscellanous
------------

### Notes

-   I was partial to [MailScanner](http://www.mailscanner.info/),
    another Perl-based interface which looks like a breeze to install.
    However, the [Postfix docs](http://www.postfix.org/addon.html) say
    it uses "unsupported methods to manipulate Postfix queue files
    directly." Okay.
-   SpamAssassin can be a bit daunting to configure. I found
    \[<http://www.yrex.com/s>
-   A big portion of configuration is setting up users for clamav,
    amavis, postfix, etc. for security. I don't have to worry about this
    given Red Hat packages, but it definitely isn't something to forget.

Errors
------

### (!)ClamAV-clamd av-scanner FAILED:... lstat() failed: Permission denied. ERROR

Add the user `clamav` to the group `amavis`.

`   usermod --groups amavis clamav`

Then add this to `/etc/amavisd.conf`

`   AllowSupplementaryGroups yes`

This might help also

`   chmod -R 775 /var/amavis/tmp`

### (!)WARN: all primary virus scanners failed, considering backups

This one's simpler. Make sure that ClamAV is running, and that you've
uncommented its definition in `/etc/amavisd.conf`

Footnotes
---------

<references />
References
----------

-   [`content_filter` in
    postconf](http://www.postfix.org/postconf.5.html#content_filter)
-   [Great overview and examples of content filtering with
    Postfix](http://www.postfix.org/FILTER_README.html)
-   [An Amavis frontend](http://myamavis.kapott.org/)

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")

[^1]: A lot of guides online talk about "injection" to Amavisd-new and
    "reinjection" back to Postfix.
