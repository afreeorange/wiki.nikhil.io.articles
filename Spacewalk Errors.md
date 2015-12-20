On Server
---------

### Invalid Password - Jabber Daemon

If you see this in `/var/log/rhn/osa-dispatcher.log`

`   2011/09/08 17:18:12 -05:00 7402 0.0.0.0: osad/jabber_lib.__init__`  
`   2011/09/08 17:18:13 -05:00 7402 0.0.0.0: osad/jabber_lib.setup_connection('Connected to jabber server', 'spacewalk.example.com')`  
`   2011/09/08 17:18:13 -05:00 7402 0.0.0.0: osad/jabber_lib.register('ERROR', 'Invalid password')`

Run this [as a cron job every
night](https://www.redhat.com/archives/spacewalk-list/2010-September/msg00078.html)
on **the server**:

`   /sbin/service jabberd stop`  
`   /sbin/service osa-dispatcher stop`  
`   rm -Rf /var/lib/jabberd/db/*`  
`   /sbin/service jabberd start`  
`   /sbin/service osa-dispatcher start`

Run this on **the client**:

`   service osad stop`  
`   rm /etc/sysconfig/rhn/osad-auth.conf`  
`   service osad start`

### Checking the FQDN quickly

All three commands should return the same thing (host.domain.tld)

``  openssl x509 -text -in `cat /etc/rhn/rhn.conf | grep osa_ssl_cert | sed 's/.* =//'` | grep '^ ``[`:space:`](:space: "wikilink")`*Subject:.*CN=' | sed 's/.*CN=//' `  
``  openssl x509 -text -in `grep '^ ``[`:space:`](:space: "wikilink")`*<id\ .*pemfile="' /etc/jabberd/c2s.xml | sed 's/.*pemfile="`\\([^"]*\\)`` .*/\1/'` | grep '^ ``[`:space:`](:space: "wikilink")`*Subject:.*CN=' | sed 's/.*CN=`\\(.*\\)`[\/$].*/\1/'`  
` grep '^`[`:space:`](:space: "wikilink")`*<id\ .*pemfile="' /etc/jabberd/c2s.xml | sed 's/.*>`\\(.*\\)`.*<.*/\1/'`

On Client
---------

### osad\_client.\_check\_signature: Signatures do not match

`   `[`Most` `likely`
`you`](http://markmail.org/message/nf2hjjlcykzu6uqk#query:+page:1+mid:upgpa2j4sqnpw6z4+state:results)` re-ran rhnreg_ks to re-register the system to your spacewalk server. `  
`   This broke the association of osad user <> systemid that is stored in the DB. `  
`   Remove the auth file for osad from within /etc/sysconfig/rhn/ on the client and `  
`   restart osad, this will re-generate a new osad auth file and this error should go away.`

So:

`  service osad stop`  
`  rm /etc/sysconfig/rhn/osad-auth.conf`  
`  service osad start`

[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
