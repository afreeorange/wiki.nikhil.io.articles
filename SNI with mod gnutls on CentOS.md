Installed on a 64-bit CentOS 5.8 system.

Configuring Apache for SNI
--------------------------

`# Configure CentOS Testing repo and install mod_gnutls`  
`wget `[`http://dev.centos.org/centos/5/CentOS-Testing.repo`](http://dev.centos.org/centos/5/CentOS-Testing.repo)` -P /etc/yum.repos.d`  
`yum install mod_gnutls`

Here's the package manifest:

` /etc/httpd/conf.d/mod_gnutls.conf`  
` /etc/httpd/conf/dhfile`  
` /etc/httpd/conf/rsafile`  
` /usr/lib64/httpd/modules/libmod_gnutls.so`  
` /usr/share/doc/mod_gnutls-0.2.0`  
` /usr/share/doc/mod_gnutls-0.2.0/LICENSE`  
` /usr/share/doc/mod_gnutls-0.2.0/NOTICE`  
` /usr/share/doc/mod_gnutls-0.2.0/README`  
` /var/cache/mod_gnutls_cache `  

Symlink the shared object file:

` ln -s /usr/lib64/httpd/modules/libmod_gnutls.so /etc/httpd/modules/mod_gnutls.so`

Disable `ssl.conf` from loading

` mv /etc/httpd/conf.d/ssl.conf{,.old}`

Edit `/etc/httpd/conf.d/mod_gnutls.conf` and uncomment (or add) the
following:

` LoadModule gnutls_module modules/mod_gnutls.so`  
` AddType application/x-x509-ca-cert .crt`  
` AddType application/x-pkcs7-crl    .crl`  
` Listen 443`

Configuring Virtual Hosts
-------------------------

Let's say I want SNI for two virtual hosts:

-   test.example.com
-   ir-devel.eng.uiowa.edu (merely a CNAME for the above)

I create a configuration file for each in `/etc/httpd/conf.d/`. Let's
start with `test.example.com`. Here's a skeleton:

` `<VirtualHost 19.65.24.170:80>  
`     ServerName test.example.com`  
`     DocumentRoot /var/www/html/test.example.com`  
`     ServerAdmin support@test.example.com`  
` `  
`     `<Directory />  
`         Options FollowSymLinks -Indexes`  
`         AllowOverride All`  
`     `</Directory>  
` `  
`     CustomLog /var/log/httpd/devel3-access.log combined`  
`     ErrorLog /var/log/httpd/devel3-error.log`  
`     LogLevel warn`  
` `</VirtualHost>  
` `  
` `<VirtualHost 19.65.24.170:443>  
`     GnuTLSEnable on`  
`     GnuTLSCertificateFile /etc/pki/tls/certs/test.example.com.crt`  
`     GnuTLSKeyFile /etc/pki/tls/private/test.example.com.key`  
` `  
`     ServerName test.example.com`  
`     DocumentRoot /var/www/html/test.example.com`  
`     ServerAdmin support@test.example.com`  
` `  
`     `<Directory />  
`         Options FollowSymLinks -Indexes`  
`         AllowOverride All`  
`     `</Directory>  
` `  
`     CustomLog /var/log/httpd/devel3-access.log combined`  
`     ErrorLog /var/log/httpd/devel3-error.log`  
`     LogLevel warn`  
` `</VirtualHost>

I do the same for the other virtual host and restart Apache. Done.

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
