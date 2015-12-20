Installation
------------

[Download the tarball](http://sourceforge.net/projects/rkhunter/files/),
extract it, and:

` ./installer.sh --layout default --install`

You can also specify `--layout RPM` instead and create an RPM. However,
you will need to export a value for the `$RPM_BUILD_ROOT` variable.
`rkhunter` installs itself as follows (on a 64-bit machine):

` INSTALLDIR=/usr/local`  
` DBDIR=/var/lib/rkhunter/db`  
` SCRIPTDIR=/usr/local/lib64/rkhunter/scripts`  
` TMPDIR=/var/lib/rkhunter/tmp`  
` USER_FILEPROP_FILES_DIRS=/etc/rkhunter.conf`

Update
------

` [root@support rkhunter-1.3.6]# rkhunter --update`  
` [ Rootkit Hunter version 1.3.6 ]`  
` `  
` Checking rkhunter data files...`  
`   Checking file mirrors.dat                                  [ No update ]`  
`   Checking file programs_bad.dat                             [ No update ]`  
`   Checking file backdoorports.dat                            [ No update ]`  
`   Checking file suspscan.dat                                 [ No update ]`  
`   Checking file i18n/cn                                      [ No update ]`  
`   Checking file i18n/de                                      [ No update ]`  
`   Checking file i18n/en                                      [ No update ]`  
`   Checking file i18n/zh                                      [ No update ]`  
`   Checking file i18n/zh.utf8                                 [ No update ]`

Configure
---------

Edit `/etc/rkhunter.conf` and make sure you have the package manager set
to RPM:

` PKGMGR=RPM`

Now create the properties file. *It is **vitally** important to do this
on a system you're **sure** hasn't been compromised.*

` rkhunter --propupd`

Now scan your system:

` rkhunter -c`

The output is sent to `/var/log/rkhunter.log`.

Other stuff
-----------

-   In case you're warned about scripts, files and directories which you
    *know* are okay, you can whitelist them with `SCRIPTWHITELIST`,
    `ALLOWHIDDENFILE`, and `ALLOWHIDDENDIR` respectively in
    `rkhunter.conf`.

<!-- -->

-   You may get warnings like these in `rkhunter.log`:

` Warning: One or more of these files were found: backdoor, adore.o, mod_rootme.so...`

This may or may not be innocuous, so it's best to check. Use the files
below.

### Quick checker script

` #!/bin/bash`  
` `  
` SUSP_FILES=$(cat suspiciousfilelist)`  
` lsof -F n -w -n | grep '^n/' | sed -e 's/^n//' | sort | uniq | grep "$SUSP_FILES"`

### Full list of files

` backdoor`  
` adore.o`  
` mod_rootme.so`  
` phide_mod.o`  
` lbk.ko`  
` vlogger.o`  
` cleaner.o`  
` cleaner`  
` ava`  
` tzava`  
` mod_klgr.o`  
` hydra`  
` hydra.restore`  
` ras2xm`  
` vobiscum`  
` sshd3`  
` system`  
` t0rnsb`  
` t0rns`  
` t0rnp`  
` rx4u`  
` rx2me`  
` crontab`  
` sshdu`  
` glotzer`  
` holber`  
` xhide`  
` xh`  
` emech`  
` psybnc`  
` mech`  
` httpd.bin`  
` mh`  
` xl`  
` write`  
` Phantasmagoria.o`  
` lkt.o`  
` nlkt.o`

Sources
-------

-   [Linux Detecting / Checking Rootkits with Chkrootkit and rkhunter
    Software](http://www.cyberciti.biz/faq/howto-check-linux-rootkist-with-detectors-software/)
-   [rkhunter installation
    notes](http://oesediez.blogspot.com/2008/06/installing-rootkit-hunter-on-centos-5.html)
-   [rkhunter RPMs on sw.be](http://packages.sw.be/rkhunter/)
-   [Detailed Installation and
    Configuration](http://sourceforge.net/apps/trac/rkhunter/wiki/SPRKH)

[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:Knowledgebase
Articles](Category:Knowledgebase_Articles "wikilink") [Category:Nikhil's
Notes](Category:Nikhil's_Notes "wikilink") [Category:From a past
sysadmin life](Category:From_a_past_sysadmin_life "wikilink")
