Configuring Channels and Repositories
=====================================

Configure channels and repositories as described [on this
page](http://wiki.centos.org/HowTos/PackageManagement/Spacewalk#head-7a284eb582941c82ecd294ebc5c16c404a242e3e).

The command executed when you schedule a sync is (for example):

` /usr/bin/python -u /usr/bin/spacewalk-repo-sync --channel epel-i386 --type yum`

-   You can then see what's happening by `tail`-ing the output of that
    repository in `/var/log/rhn/reposync`. This is very helpful when
    you're trying to diagnose issues.
-   I couldn't get two syncs to take place simultaneously.

Syncing repositories manually
-----------------------------

Scheduling syncs via the Spacewalk server will almost always require you
to tail files in `/var/log/rhn/reposync` and will produce a strange
directory structure (see sections below). However, if you wanted to do
these things yourself, you could try this:

` # Sync repositories to a local folder`  
` reposync –arch=x86_64 -p /var/www/html/pub/CentOS5-x86_64 -d -l -g -n -q –repoid=base64       > /dev/null`  
` reposync –arch=x86_64 -p /var/www/html/pub/CentOS5-x86_64 -d -l -g -n -q –repoid=updates64    > /dev/null`  
` reposync –arch=x86_64 -p /var/www/html/pub/CentOS5-x86_64 -d -l -g -n -q –repoid=extras64     > /dev/null`  
` reposync –arch=x86_64 -p /var/www/html/pub/CentOS5-x86_64 -d -l -g -n -q –repoid=centosplus64 > /dev/null`  
` reposync –arch=x86_64 -p /var/www/html/pub/CentOS5-x86_64 -d -l -g -n -q –repoid=epel64       > /dev/null`  
` reposync –arch=x86_64 -p /var/www/html/pub/CentOS5-x86_64 -d -l -g -n -q –repoid=spacewalk-client-tools64 > /dev/null`  
` `  
` # Make the Spacewalk server aware of synced repos`  
` rhnpush –channel=centos5basex86_64          –username=rhnusername –password=rhnpassword –server=`[`http://localhost/APP`](http://localhost/APP)` –dir=/var/www/html/pub/CentOS5-x86_64/base/CentOS `  
` rhnpush –channel=centos5updates64           –username=rhnusername –password=rhnpassword –server=`[`http://localhost/APP`](http://localhost/APP)` –dir=/var/www/html/pub/CentOS5-x86_64/updates/RPMS`  
` rhnpush –channel=centos5extrasx86-64        –username=rhnusername –password=rhnpassword –server=`[`http://localhost/APP`](http://localhost/APP)` –dir=/var/www/html/pub/CentOS5-x86_64/extras/RPMS`  
` rhnpush –channel=centos5plusx86_64          –username=rhnusername –password=rhnpassword –server=`[`http://localhost/APP`](http://localhost/APP)` –dir=/var/www/html/pub/CentOS5-x86_64/centosplus/RPMS`  
` rhnpush –channel=spacewalkclienttoolsx86_64 –username=rhnusername –password=rhnpassword –server=`[`http://localhost/APP`](http://localhost/APP)` –dir=/var/www/html/pub/CentOS5-x86_64/spacewalk-client-tools`  
` rhnpush –channel=epel5x86_64                –username=rhnusername –password=rhnpassword –server=`[`http://localhost/APP`](http://localhost/APP)` –dir=/var/www/html/pub/CentOS5-x86_64/epel`

Determing GPG information
-------------------------

Adding a channel requires the key URL, ID and fingerprint. This is easy
to determine:

` wget `[`http://dev.centos.org/centos/RPM-GPG-KEY-CentOS-testing`](http://dev.centos.org/centos/RPM-GPG-KEY-CentOS-testing)  
` gpg --import RPM-GPG-KEY-CentOS-testing`  
` gpg --list-public-keys --fingerprint`

This will produce output like:

` /root/.gnupg/pubring.gpg`  
` ------------------------`  
` pub   1024D/F24F1B08 2002-04-23 [expired: 2004-04-22]`  
`       Key fingerprint = D8CC 06C2 77EC 9C53 372F  C199 B1EE 1799 F24F 1B08`  
` uid                  Red Hat, Inc (Red Hat Network) `<rhn-feedback@redhat.com>  
` `  
` pub   2048R/C236FD2B 2010-12-20 [expires: 2011-12-20]`  
`       Key fingerprint = 8E05 7113 DF16 CB7A E7A5  0422 A8E4 0177 C236 FD2B`  
` uid                  Nikhil Anand `<anand.nikhil@gmail.com>  
` `  
` pub   1024D/910620BF 2010-05-12`  
`       Key fingerprint = B3B6 A608 6012 F724 52C3  03F4 D085 AAC6 9106 20BF`  
` uid                  Nicolai `<nicolai@chocolatine.org>  
` sub   4096g/29673670 2010-05-12`  
` `  
` pub   1024D/7203F491 2005-11-19`  
`       Key fingerprint = BCD0 0AEB A3C0 39D7 25E0  663C 5C37 C0B1 7203 F491`  
` uid                  CentOS-testing (CentOS Developers testing key) `<centos@centos.org>  
` sub   2048g/537F5CB3 2005-11-19`

`7203F491` is your key ID.

Local repositories and Log files
--------------------------------

RPMs are staged in `/var/cache/reposync` and then moved to
`/var/satellite/redhat`. The directory structure looks like this:

` [root@spacewalk /var/satellite/redhat/1]# tree 62c`  
` 62c`  
` |-- hmaccalc`  
``  |   `-- 0.9.6-3.el5 ``  
``  |       `-- i386 ``  
``  |           `-- 62cdfcfe805ee49082434653625f84f4 ``  
``  |               `-- hmaccalc-0.9.6-3.el5.i386.rpm ``  
` |-- python-docs`  
``  |   `-- 2.4.3-1.1 ``  
``  |       `-- noarch ``  
``  |           `-- 62cbc246046f1cb5306758842f738725 ``  
``  |               `-- python-docs-2.4.3-1.1.noarch.rpm ``  
``  `-- tkinter ``  
``      `-- 2.4.3-27.el5 ``  
``          `-- i386 ``  
``              `-- 62c1a8dc30931e7ec0d947dbef6db2d7 ``  
``                  `-- tkinter-2.4.3-27.el5.i386.rpm ``

Log files are stored in `/var/log/rhn`. When you schedule a sync action,
you'll see log files appear in `/var/log/rhn/reposync`. For other
actions, use `rhn_taskomatic_daemon.log` (use `ls -ltr` to see which log
files have changed after you've done something!)

Syncing Errata
==============

[This page](http://www.bioss.ac.uk/staff/davidn/spacewalk-stuff/) has a
fantastic Python script that goes through mail archives, digests and
mailing list websites for errata and pushes them to the Spacewalk
server. It does have a few limitations you should be aware of (on the
download page). It takes care of duplicates and takes *a considerable*
amount of time.

Also remember that you won't get the current month's errata this way.
The gzipped archives are only available at the end of every month from
the CentOS lists.

The script attempts to pull information on a given package using its ID.
If this fails, it looks at `package_dir` (see below). The **problem** is
that it expects `package_dir` to be a flat directory with all the RPMs
in it. This is not the default case.

I run this command daily. Not appending the `--password` option results
in the script asking for a RHN password.

` /opt/spacewalk-errata/centos-errata.py --config=/opt/spacewalk-errata/centos-errata.cfg \`  
` --password="XXXXXXXXX" \`  
` `**`--format=mail-archive.com`**

You can also write a small script that `gunzip`'s archive files from
[the actual mailing
list](http://lists.centos.org/pipermail/centos-announce/). Here's a
sample script:

` #!/bin/sh`  
` # Processes CentOS Errata and imports it into Spacewalk`  
` `  
``  DATE=`date +"%Y-%B" +d '1 month ago'` ``  
` `  
` # Fetch and uncompress errata data from the CentOS lists`  
` wget -P /opt/spacewalk-errata/errata `[`http://lists.centos.org/pipermail/centos-announce/$DATE.txt.gz`](http://lists.centos.org/pipermail/centos-announce/$DATE.txt.gz)  
` gunzip -f /opt/spacewalk-errata/errata/$DATE.txt.gz`  
` `  
` # Processes and imports the errata.`  
` cd /opt/spacewalk-errata/ && \`  
` /opt/spacewalk-errata/centos-errata.py --format=archive /opt/spacewalk-errata/errata/$DATE.txt \`  
` --config=/opt/spacewalk-errata/centos-errata.cfg >> /var/log/centos-errata.log`

I don't know why you have to supply your password; it should already be
in the config file (`/opt/spacewalk-errata/centos-errata.cfg`). Speaking
of, here's what mine looks like:

` [centos errata]`  
` #Required to identify applicable messages on the centos-announce mailing list`  
` version=5`  
` `  
` #Useful for interpolation below, not used by tool itself`  
` release=6`  
` `  
` #If true the script will attempt to use the Redhat Network to populate the errata description`  
` scrape_rhn=False`  
` `  
` # I only set spacewalk and not "dir" since I want the script to rely on Spacewalk `  
` # exclusively to get package signatures`  
` search_strategies=spacewalk`  
` `  
` #Maximum number of errata to process at once. Only relevant to format 'mail-archive.com'`  
` #max_errata`  
` `  
` [spacewalk]`  
` server=spacewalk.eng.uiowa.edu`  
` login=admin`  
` #The tool will prompt you if you don't specify a password`  
` password=XXXXXXXX`  
` `  
` [i386]`  
` # Enter the name of the channel that the errata will link to.`  
` channel=centos-5.5-i386-updates`  
` `  
` [x86_64]`  
` # Enter the name of the channel that the errata will link to.`  
` channel=centos-5.5-x86_64-updates`

(Stateful) Firewall Rules
=========================

You'll have to accept incoming connections on port 443 (HTTPS) for basic
functionality. If you want to push configs to clients, here are the
relevant stateful `iptables` rules. Port 5222 shows up in
`/etc/services` as "xmpp-client".

` # On server`  
` iptables -A INPUT -p tcp --dport 5222 -m state --state NEW -j ACCEPT`  
` iptables -A INPUT -p udp --dport 5222 -m state --state NEW -j ACCEPT`  
` `  
` # If you're filtering outputs on client`  
` iptables -A OUTPUT -d $SPACEWALK_SERVER -p tcp --dport 5222 -m state --state NEW -j ACCEPT`  
` iptables -A OUTPUT -d $SPACEWALK_SERVER -p udp --dport 5222 -m state --state NEW -j ACCEPT`

Error with `repodata.xml` with EPEL
===================================

For some reason `/var/cache/rhn/repodata/epel-i386` doesn't have the
`repodata.xml` file. The source I configured the repository with doesn't
have it either. I had to manually download it:

` wget -P /var/cache/rhn/repodata/epel-i386/ `[`http://linux.mirrors.es.net/fedora-epel/5/i386/repodata/repomd.xml`](http://linux.mirrors.es.net/fedora-epel/5/i386/repodata/repomd.xml)

Pertinent services
==================

` Monitoring      `  
` MonitoringScout `  
` cobblerd        `  
` jabberd         `  
` oracle-xe       `  
` osa-dispatcher  `  
` rhn-search      `  
` taskomatic      `  
` tomcat5         `

`xinetd` and `tftpd` need to be started if you plan on kickstarting
nodes. `jabberd` is very essential to push configs to nodes.

Enable Monitoring
=================

-   Go to Admin &gt; Spacewalk Configuration &gt; Monitoring and check
    "Enable Monitoring Scout".
-   You'll need to restart the RHN server after this.

Now you need to configure each client. See the appropriate section in
the client config page for how to do this. Essentially, you'll use a
keyless SSH login as user `nocpulse` (a company [acquired by Red
Hat](http://www.crn.com/news/applications-os/18828935/red-hat-set-to-acquire-nocpulse.htm;jsessionid=IzcWoAcIx174S0nNiEbNMw**.ecappj02))
to get metrics from clients.

Although the default port for NOCpulse is 4545, you can monitor via port
22 as well. Just look for the port option when creating a probe. You can
test a connection by issuing this from the RHN server:

` ssh -l nocpulse -p 4545 -i /var/lib/nocpulse/.ssh/nocpulse-identity client.com`

Getting ready to register clients
=================================

-   Go to "Systems &gt; Activation Keys" and generate a new key.
-   It's a good idea to do this *after* setting up your base channels.
    For example, I created two keys for 32-bit and 64-bit systems with
    their respective channels as base channels (i.e. *not*
    "Spacewalk Default").
-   Make sure you check "Provisioning" if you want to push centralized
    files to your systems.
-   See the section above on monitoring; have the public key ready!

Working with the Oracle XE Database
===================================

Some quick points:

-   The 'actual' database files are installed in `/usr/lib/oracle/xe`
    for backups.
-   If unset, the `$ORACLE_HOME` variable should point at
    `/usr/lib/oracle/xe/app/oracle/product/10.2.0/server/`
-   You should have a script which uses Oracle's RMAN to back up your
    database:

` $ORACLE_HOME/config/scripts/backup.sh`

-   There's also another one, but it merely opens up the first script in
    X11:

` $ORACLE_HOME/config/scripts/backupdatabase.sh`

You can do a:

-   'Cold' backup: merely `rsync` the files keeping the database
    *offline*
-   'Hot' backup: use the scripts above, done when DB is *online*
-   Manual backup with the web interface (tedious)

I personally do a 'cold' backup since I couldn't get `startup mount` to
work with setting up `backup.sh`.

Weird Fonts in History Graphs
=============================

![Example of screwed up
fonts](ProbeGraph.do.png "Example of screwed up fonts")

Spacewalk uses jFreeChart for graphing. jFreeChart relies on the JVM for
font configuration. This is found in the `$JAVAHOME/lib/fontconfig.*`
files (there's [more
information](http://download.oracle.com/javase/1.5.0/docs/guide/intl/fontconfig.html)
on this).

Basically, you need to install [the
DejaVu](http://dejavu-fonts.org/wiki/Main_Page) font package on your
server if you see weird, cursive fonts:

` yum -y install dejavu-lgc-fonts`

Cleaning your log files
=======================

`reposync` generates a lot of logs and doesn't have a `logrotate`
configuration. So I added this to `crontab` to prevent things from
getting out of control:

` # Clean the log directory every day at noon`  
` 0 0 * * * /usr/bin/find /var/log/rhn/reposync/ -type f -ctime +0 | xargs rm -rf`

Sources
=======

-   [The CentOS mirror
    list](http://www.centos.org/modules/tinycontent/index.php?id=30)
-   [32-bit Spacewalk Server with 64-bit
    Clients](http://thedonkeyland.com/blog/2009/07/32-bit-spacewalk-server-with-64-bit-clients-centos/)
-   [Spacewalk
    Setup](http://translate.google.com/translate?hl=en&sl=nl&tl=en&u=http://www.falsyana.com/linux/spacewalk/)
    (in Dutch but *ridiculously* helpful)
-   [Backing up](http://www.markcallen.com/oracle/oracle-xe-backup) and
    [recovering](http://www.markcallen.com/oracle/oracle-xe-recovery) an
    Oracle XE database
-   [A *fantastic* overview of Oracle DB
    backups](http://www.thefreyers.net/doku.php?id=technology:databases:how_to_backup_an_oracle_xe_database)
-   [Oracle's documentation on Backing up and Restoring
    DBs](http://download.oracle.com/docs/cd/B25329_01/doc/admin.102/b25107/backrest.htm#CIHEHCEA)
-   [Script to import centos-announce messages into a spacewalk
    server](http://www.bioss.ac.uk/staff/davidn/spacewalk-stuff/)
    -   [Configuring SpaceWalk to import
        Errata](http://www.misdivision.com/blog/configuring-spacewalk-to-import-centos-errata)
-   [Some statistics on Spacewalk](http://blog.delouw.ch/tag/spacewalk/)
-   [Jabber and
    OSAD](https://fedorahosted.org/spacewalk/wiki/JabberAndOSAD)
-   [Centralized system management with Spacewalk
    1.3](http://www.linux-specialist.nl/2011/02/centralized-system-management-with-spacewalk/)
    (dude claims package deployment doesn't work...)
    -   [Scheduled Package Installs
        Failing](https://www.redhat.com/archives/spacewalk-list/2011-February/msg00180.html)
        (dude upgrades from 1.2 to 1.3, has checklist of how he got
        package management to work)
-   [RedHat's package-signing
    keys](https://access.redhat.com/security/team/key/)
-   [Step-by-step instructions on how to enable
    monitoring](https://bugzilla.redhat.com/show_bug.cgi?id=492177#c0)
    -   [Official Red Hat documentation on the monitoring
        entitlement](https://rhn.redhat.com/rhn/help/reference/rhn370/en/s1-mon-rhnmd.jsp)
-   [Changing Spacewalk Server's self-signed
    certificates](http://unfuckablelinux.com/2008/07/02/spacewalk-and-avoiding-self-signed-certificates/)

[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
