Set up with [Postfix](Postfix "wikilink"), Postgrey,
[Dovecot](Dovecot "wikilink"), Cyrus SASL, [ClamAV](ClamAV "wikilink"),
[SpamAssassin](SpamAssassin "wikilink"),
[Amavisd-new](Amavisd-new "wikilink"), and
[Fail2Ban](Fail2Ban_for_Dovecot "wikilink") on CentOS 6.4.

Here's [a great guide](http://www.shisaa.jp/postset/mailserver-1.html)
([cached](:File:Shisaa_-_Mailserver.pdf "wikilink")).

### Billy Gorbachev's options:

-   OpenBSD. Eliminates need for fail2ban, since pf has this
    functionality built in.

<!-- -->

-   qmail, not Postfix.

<!-- -->

-   OpenBSD spamd, not Postgrey. spamd hurts spammers more, and hurts
    legit senders less. logs are hilarious, e.g. 178.33.129.117:
    disconnected after 3995 seconds. lists: spamd-greytrap

<!-- -->

-   Replace spamassassin with Spamhaus hooks in qmail (tcpserver).

<!-- -->

-   qmail-pop3d, not Dovecot. Dovecot has a poor security track record.
    Download mail over (pop3 over CurveCP), using fetchmail + CurveCP
    command-line tools. Store locally in Maildir, backup
    as necessary/desired.

<!-- -->

-   drop ClamAV and Amavisd. Run FreeBSD or OpenBSD on your desktop.

<!-- -->

-   Add GnuPG to your local mail client.

<!-- -->

-   Add Spamhaus DROP and eDROP to network edge pf tables
    <http://www.spamhaus.org/drop/>

<!-- -->

-   Add a TXT SPF record for your domain, including your servers,
    followed by "-all"

Several of these changes can be made (SPF record, DROP, GnuPG) without
modifying your existing setup.



