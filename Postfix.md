Written for CentOS 6.4, Postfix 2.6.6

Overview
--------

Things were easier to set up after understanding these things, even
cursorily.

### Mail Transfer - Basic Idea

Lots of formal abbreviations!

`   Sender > Server > Server > ... > Server > Receiver`  
`   (MUA)    (MTA)    (MTA)          (MTA)     (MUA)`

You can be a bit more granular:

`   Sender  >  Server > Server > ... > Server     >     Receiver`  
`   (MUA > MSA > MTA)    (MTA)          (MTA > MDA > MRA > MUA)`

This separation of purpose is good since you can use variety of
applications at each stage. Lot of possibilities. E.g.

`   Sender > Postfix > Procmail > Clam Anti-Virus > SpamAssassin > Procmail > Receiver`  
`   (MUA)    (MTA)     (                       MDA                        )     (MUA)`

### Open Relays

Where it is not required to (1) authenticate to your server, and/or (2)
be in the same network as the server to send email. This is very bad for
public-facing mail servers. From a simpler time when there were very few
email servers and everybody was nice to each other.

Installation
------------

`   yum install postfix`  
`   ln -s /usr/sbin/sendmail.postfix /etc/alternatives/mta --force`

Configuration
-------------

Working with `/etc/postfix/main.cf`.

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
