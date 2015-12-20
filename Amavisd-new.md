For CentOS 6.4, with SpamAssassin 3.3, Amavisd-new 2.8, and ClamAV 0.97

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

`   # Start`  
`   chkconfig amavisd on; service amavisd start`  
`   chkconfig clamd on; service clamd start`  
`   chkconfig spamassassin on; service spamassassin start`

`   # Update`  
`   freshclam`  
`   sa-update`

You'll have to update only once; both ClamAV and SpamAssassin come with
their own `cron` jobs. Handy.

Configuration
-------------

Miscellanous
------------

-   I was partial to [MailScanner](http://www.mailscanner.info/),
    another Perl-based interface which looks like a breeze to install.
    However, the [Postfix docs](http://www.postfix.org/addon.html) say
    it uses "unsupported methods to manipulate Postfix queue files
    directly." Okay.
-   SpamAssassin can be a bit daunting to configure. I found [this
    online
    configurator](http://www.yrex.com/spam/spamconfig.php) helpful.

Footnotes
---------

References
----------

-   <http://myamavis.kapott.org/>

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")

[^1]: A lot of guides online talk about "injection" to Amavisd-new and
    "reinjection" back to Postfix.
