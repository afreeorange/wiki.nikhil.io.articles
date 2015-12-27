[TOC]

Preliminary Notes
-----------------

I tested SpaceWalk on a 64-bit CentOS 5.5 system as the server and a
32-bit CentOS 5.4 system as the client. 

Pre-Flight
----------

### Some Pre-requisites

    rpm -ivh http://spacewalk.redhat.com/yum/1.3/RHEL/5/x86_64/spacewalk-client-repo-1.3-1.el5.noarch.rpm
    rpm -ivh http://spacewalk.redhat.com/yum/1.3/RHEL/5/x86_64/spacewalk-repo-1.3-1.el5.noarch.rpm
    wget -P /etc/yum.repos.d http://jpackage.org/jpackage.repo
    yum -y install bc glibc libaio httpd-devel tomcat5-webapps tomcat5-admin-webapps  
    yum -y install rlwrap --enablerepo=epel

### Import the RedHat GPG Key

    wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release http://www.redhat.com/security/37017186.txt

### Install and Configure Oracle XE

You'll need [Oracle XE](http://www.oracle.com/technetwork/database/express-edition/downloads/102xelinsoft-102048.html)
and the [InstantClient](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html).
Oracle thankfully provides [a 32-bit version of InstantClient](http://www.oracle.com/technetwork/topics/linuxsoft-082809.html)
as well.

*   XE has 32-bit installers
    *   I used `oracle-xe-univ-10.2.0.1-1.0.i386.rpm`
*   InstantClient has 32 and 64-bit installers
    *   I installed
        `oracle-instantclient11.2-basic-11.2.0.2.0.x86_64.rpm` and
        `oracle-instantclient11.2-sqlplus-11.2.0.2.0.x86_64.rpm`

Then execute `/etc/init.d/oracle-xe configure`. I used defaults (port
8080 for Oracle Application Express, port 1521 for the database
listener.) You can now go to `http://127.0.0.1:8080/apex` to access the
database; use `system` for the username and the password you specified
at install.

### Errors with `sqlplus`

**Please note** that `sqlplus` is `sqlplus64` on (drumroll) 64-bit
systems. This being said, you'll get the following error when you try
connecting to the DB via a terminal:

    [root@support spacewalk]# sqlplus64 'sys/password@//localhost/XE as sysdba'  
    sqlplus64: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory

The issue is that you need to add `libsqlplus.so` to PATH:

    [root@support spacewalk]# updatedb  
    [root@support spacewalk]# locate libsqlplus.so  
    /usr/lib/oracle/11.2/client64/lib/libsqlplus.so  
    /usr/lib/oracle/xe/app/oracle/product/10.2.0/server/lib/libsqlplus.so  
    [root@support ~]# # Add the above to PATH  
    [root@support ~]# export PATH="${PATH}:/usr/lib/oracle/11.2/client64/bin"  
    [root@support ~]# ORACLE_HOME=/usr/lib/oracle/11.2  
    [root@support ~]# export LD_LIBRARY_PATH=/usr/lib/oracle/11.2/client64/lib

Now retry the connection. It should work.

### Add the SpaceWalk user

    # sqlplus 'sys/YOUR_PASSWORD@//localhost/XE as sysdba'  
    SQL> create user spacewalk identified by spacewalk default tablespace users;  
    SQL> grant dba to spacewalk;  
    SQL> quit

Test this; you should get a login.

    sqlplus spacewalk/spacewalk@//localhost/XE

### Increase the number of simultaneous connections to the DB

The Oracle XE installation guide has [an explanation of why this is
necessary](https://fedorahosted.org/spacewalk/wiki/OracleXeSetup).

    SQL> alter system set processes = 400 scope=spfile;   
    SQL> alter system set "_optimizer_filter_pred_pullup"=false scope=spfile;   
    SQL> alter system set "_optimizer_cost_based_transformation"=off scope=spfile;   
    SQL> quit

Restart the Oracle XE service by issuing `/sbin/service oracle-xe restart`. If 
you get any errors, see the installation guide linked to above.

### Add the Oracle Service Definition

Add this to `/etc/tnsnames.ora`

```lisp
XE =
    (DESCRIPTION =  
       (ADDRESS_LIST =  
          (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))  
       )  
       (CONNECT_DATA =  
          (SERVICE_NAME = xe)  
       )  
    )
```

Install and Verify Tomcat
-------------------------

If you think the installation's going peachy right now, you're wrong.

### Install the `mod_jk` module

Apache needs to be configured with the `mod_jk` module. `mod_jk` is
available from [the `jpackage-rhel` repo](http://jpackage.org/jpackage.repo). 
*However*, version 2.0 of `mod_jk` will not work with version 2.2 of Apache. 
Yay OSS!

This means that it will need to be compiled. I grabbed [version 1.2.31](http://apache.mirrors.hoobly.com/tomcat/tomcat-connectors/jk/source/jk-1.2.31/tomcat-connectors-1.2.31-src.tar.gz)
and compiled it. Keep in mind that you need to do this with
`./configure --with-apxs=/usr/sbin/apxs` else it will complain about not
finding "apache" or "netscape". Compilation will install `mod_jk` in
`/usr/lib64/httpd/modules/mod_jk.so`. I've also [archived the 32 and
64-bit versions](http://support.eng.uiowa.edu/software/archive/mod_jk22/).

Now add this to `/etc/httpd/conf/httpd.conf` and restart Apache.

    LoadModule jk_module modules/mod_jk.so

### Modifying Tomcat's Default Port

Consider Tomcat's default ports:

| Port |                                     Purpose                                      |
|------|----------------------------------------------------------------------------------|
| 8005 | "Shutdown" port                                                                  |
| 8007 | Replies to [AJP](http://en.wikipedia.org/wiki/Apache_JServ_Protocol) 12 requests |
| 8009 | Replies to AJP 13 requests (SpaceWalk needs this)                                |
| 8080 | Standard HTTP connector                                                          |

But we configured port 8080 for the Oracle XE database! If you tailed
the output of `/var/log/tomcat5/catalina.out`, you'll see a "8080
already in use" message (sort of). To change this port to something like
8081, edit `/etc/tomcat5/server.xml`, search for 8080 and change it:

    <Connector port="8081" maxHttpHeaderSize="8192" maxThreads="150"....

Now stop Apache, restart Tomcat, and check the output of `netstat`:

    [root@support ~]# netstat -tulpn | grep java | grep LISTEN  
    tcp        0      0 127.0.0.1:32000             0.0.0.0:*                   LISTEN      3763/java             
    tcp        0      0 ::ffff:127.0.0.1:8005       :::*                        LISTEN      5871/java             
    tcp        0      0 ::ffff:127.0.0.1:8009       :::*                        LISTEN      5871/java             
    tcp        0      0 ::ffff:127.0.0.1:2828       :::*                        LISTEN      3763/java             
    tcp        0      0 ::ffff:127.0.0.1:8081       :::*                        LISTEN      5871/java

Yay! Now stop Apache, restart Tomcat, start Apache, and you should be
good to go. If you didn't change this, you'd see an error like:

    [Tue Feb 22 11:49:09 2011] [error] (111)Connection refused: proxy: AJP: attempt to connect to 127.0.0.1:8009 (*) failed

Install SpaceWalk
-----------------

    yum install spacewalk-oracle

This will pull down everything necessary to run SpaceWalk. If you get
depsolving errors, try enabling the EPEL repo (if you've disabled it.)

    yum install spacewalk-oracle --enablerepo=epel

Grab a cup of tea. This will take a while. Then run the setup binary:

    spacewalk-setup --disconnected

Use "XE" (sans quotes) for the database SID and spacewalk/spacewalk for
the username/password. Don't set up `cobbler` to use `tftp` and `xinetd`
just yet. Once installation is complete, navigate to the *'https*
version of your site to launch SpaceWalk. Remember that SpaceWalk
requires a FQDN all to itself (on port 443 at least).

Errors
------

### Oracle XE Errors

Oracle XE is probably the jankiest part of your Spacewalk install. You
might see these errors when trying the `sqlplus` command with the
`spacewalk` or `sys` users:

    ORA-12516: TNS:listener could not find available handler with matching protocol  
    stack  
    ORA-12514: TNS:listener does not currently know of service requested in connect  
    descriptor  
    ORA-12541: TNS: no listener

I couldn't find anything pertinent other than the formatting and
permissions of the `/etc/tnsnames.ora` file. Shutting down the Oracle XE
instance, Tomcat and Apache, and then starting them up (in that order)
seemed to help. Sometimes, a mere restart of the `oracle-xe` service
seemed to help. Also:

*   Make sure that root:tomcat owns `/etc/tnsnames.ora` with 755
    *   *It is claimed that the new version of Spacewalk (1.3) doesn't
        even need the `tnsnames.ora` file...*
*   Pertinent log files are in `/var/log/rhn` and `/var/log/tomcat5`
*   [This
    page](http://www.oracledistilled.com/oracle-database/troubleshooting/errors/ora-12514-tnslistener-does-not-currently-know-of-service-requested-in-connect-descriptor/)
    says you should watch out for typos.
*   You may want to leave out the "simultaneous connections" SQL
    statements

To reconfigure the Oracle instance:

    service oracle-xe stop  
    rm -rf /etc/sysconfig/oracle-xe /var/tmp/.oracle  
    rpm2cpio oracle-xe-univ-10.2.0.1-1.0.i386.rpm | \  
      ( cd / && cpio -iud ./usr/lib/oracle/xe/app/oracle/product/10.2.0/server/config/scripts/DatabaseHomePage.sh \  
                          ./usr/lib/oracle/xe/app/oracle/product/10.2.0/server/config/scripts/postDBCreation.sql \  
                          ./usr/lib/oracle/xe/app/oracle/product/10.2.0/server/config/scripts/readonlinehelp.sh \  
                          ./usr/lib/oracle/xe/app/oracle/product/10.2.0/server/network/admin/listener.ora \  
                          ./usr/lib/oracle/xe/app/oracle/product/10.2.0/server/network/admin/tnsnames.ora \  
                          ./usr/lib/oracle/xe/app/oracle/product/10.2.0/server/config/seeddb/xeseed.dfb )  
    /etc/init.d/oracle-xe configure

**If nothing else** [read this page](https://fedorahosted.org/spacewalk/wiki/OracleXeSetup)
thoroughly.

Changing the SSL certificate
----------------------------

As you probably know, Spacewalk requires a FQDN all to itself *on port
443* (you can run other stuff on port 80). During install, you'll see a
CA generation and certificate signing process for the `https://` site.
This is a change made to `/etc/httpd/conf.d/ssl.conf`.

Since I use my own root CA to sign certificates, I tried generating a
certificate for the Spacewalk site and editing `ssl.conf` to use this
certificate. This, however, caused problems on Spacewalk clients which
rejected the certificate.

**Moral of the story**: Don't mess with `ssl.conf`.

Uninstallation
--------------

```bash
#!/bin/bash
ARCH=$(uname -i)

# Uncomment to remove Oracle XE
echo -e ">>> Removing Oracle" && sleep 2
service oracle-xe stop
rpm -e oracle-instantclient11.2-basic-11.2.0.2.0.$ARCH --nodeps
rpm -e oracle-instantclient11.2-sqlplus-11.2.0.2.0.$ARCH --nodeps
rpm -e oracle-xe-univ-10.2.0.1-1.0.i386 --nodeps
rm -rf /usr/lib/oracle
rm -rf /var/tmp/.oracle
rm -rf /root/oradiag_root/
rm -rf /etc/tnsnames.ora
rm -rf /etc/sysconfig/oracle-xe
rm -rf /etc/ld.so.conf.d/oracle*
echo "" > /etc/oratab
userdel oracle
groupdel dba

echo -e ">>> Stopping the Spacewalk Service" && sleep 2
/usr/sbin/spacewalk-service stop

echo -e ">>> Removing Spacewalk" && sleep 2
rpm -e spacewalk-repo
rpm -e spacewalk-client-repo
yum -y remove *spacewalk* jabberd *oracle*
rm -rf /var/www/html/pub
rm -rf /root/ssl-build
rm -Rf /var/lib/jabberd/db/*
rm -rf /etc/yum.repos.d/*jpackage*
rm -rf /etc/jabberd/*

echo -e ">>> Miscellaneous (EPEL, logfiles)" && sleep 2
rpm -e epel-release
rm -Rf /var/log/rhn/*

echo -e ">>> Done!"
echo -e ">>> Please clean up your crontab\n"
```

Miscellaneous Notes (from May 2009)
-----------------------------------

### Bolting down Tomcat

Did yum install tomcat5-webapps tomcat5-admin-webapps This enabled the
administration and management interfaces. User profiles found in
$CATALINA\_HOME/conf/tomcat-users.xml You need to change this file AND
INCLUDE THE ADMIN ROLE to access the "Tomcat Administration" page AND
THE MANAGER ROLE to access the "Tomcat Manager" page

    <tomcat-users>  
    <role rolename="admin" />  
    <role rolename="manager"/>  
        <user name="admin" password="t0mcaaat" roles="admin,manager" />  
    </tomcat-users>

Sources
-------

* https://fedorahosted.org/spacewalk/wiki/OracleXeSetup
* https://fedorahosted.org/spacewalk/wiki/HowToInstall
* http://www.cs.rug.nl/~jurjen/ApprenticesNotes/ch11.html
