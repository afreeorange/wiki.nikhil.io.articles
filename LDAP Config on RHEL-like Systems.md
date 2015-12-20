Client setup
------------

` authconfig  --enableldap \`  
`             --enableldapauth \`  
`             --enableldaptls \`  
`             --ldapserver='`[`ldap://directory.example.com/`](ldap://directory.example.com/)`' \`  
`             --ldapbasedn='dc=example,dc=edu' \`  
`             --enablemkhomedir \`  
`             --enableshadow \`  
`             --enablelocauthorize \`  
`             --update`

[From
here.](http://www.syntaxtechnology.com/2009/09/openldap-on-centos-5-3-part-3-the-client/)
StartTLS will be enabled for each lookup. The command above modifies
three files:

` /etc/nsswitch.conf`  
` /etc/ldap.conf`  
` /etc/openldap/ldap.conf`

Problems
--------

### Can't change passwords

` New UNIX password: `  
` Retype new UNIX password: `  
` LDAP password information update failed: Constraint violation`  
` invalid password syntax - passwords with storage scheme are not allowed`

**Disable SELinux**. Another problem could be that the passwords are
[hashed before they're
sent](http://www.redhat.com/archives/fedora-directory-users/2009-September/msg00051.html).
They need to be protected with SSL/TLS and sent in the clear.

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
