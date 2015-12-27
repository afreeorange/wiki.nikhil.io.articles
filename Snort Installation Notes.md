I installed this on a 32-bit CentOS 5.6 system, with MySQL as the output
backend for snort.

Installation
------------

### Install Snort

Snort v2.9+ use `libpcap` 1.0+. Unfortunately, CentOS 5.x still uses
v0.9.4. I tried (the hard way) to compile my own RPMs and stop swearing
at the screen. However, a gentleman by name Vincent Cojot has already
done [some awesome
heavy-lifting](http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/)
for all of us. Of course, the makers of Snort recommend that you compile
it and don't vouch for Vincent RPMs. But I have shit to do.

    mkdir snort; cd snort  
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/daq-0.5-9.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/daq-debuginfo-0.5-9.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/libdnet-1.12-7.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/libdnet-debuginfo-1.12-7.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/libdnet-devel-1.12-7.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/libdnet-progs-1.12-7.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/libpcap1-1.1.1-9.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/libpcap1-debuginfo-1.1.1-9.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/libpcap1-devel-1.1.1-9.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/snort-2.9.0.5-12.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/snort-debuginfo-2.9.0.5-12.el5.i386.rpm
    wget http://vscojot.free.fr/dist/snort/snort-2.9.0.5/RHEL5/i386/snort-mysql-2.9.0.5-12.el5.i386.rpm
    rpm -ivh *.rpm

**Important**: v2.9.0.5 RPMs have some
significant path-related issues when trying to find rulesets. I
recommend sticking with v2.9.0.4

### Install Snort Rules

You'll need to register on Snort's website before you can download a
ruleset. New rulesets are released to subscribers ($$$) 30 days before
registered users (FREE!) can download them.

    # The ruleset is a tarbomb  
    mkdir snort-rules; cd snort-rules  
    wget https://s3.amazonaws.com/snort-org/www/rules/20110329/snortrules-snapshot-2903.tar.gz
    tar -xvzf snortrules-snapshot-2903.tar.gz  
    cp etc/* /etc/snort/  
    cp rules/* /etc/snort/rules/  
    cp so_rules/precompiled/Centos-5-4/i386/2.9.0.3/* /etc/snort/so_rules/  
    cp preproc_rules/* /etc/snort/preproc_rules/

Now we're ready to bend Snort to our whim.

### Set up the Snort user and group

*The RPM should already set this up for you*. If not,

    [root@snort usr]# groupadd snort  
    [root@snort usr]# useradd -g snort snort  
    [root@snort usr]# id snort  
    uid=505(snort) gid=506(snort) groups=506(snort)

You'll then need to `chown` `/var/log/snort`.

### Set up the MySQL database

Set up your database (call it "snort") and user (also called "snort")
and password ("PASSWORD"). Now load the schema:

    mysql -uroot -p snort < /usr/share/snort-2.9.0.5/schemas/create_mysql

Configuring Snort
-----------------

Start editing `/etc/snort/snort.conf` (make a copy of the original
first!) Here are my modifications to the original file:

```bash
ipvar HOME_NET 19.25.78.0/24  
portvar SSH_PORTS [22,3232]  
portvar HTTP_PORTS [80,443,8080,9066]  
  
# These were relative  
var RULE_PATH /etc/snort/rules  
var SO_RULE_PATH /etc/snort/so_rules  
var PREPROC_RULE_PATH /etc/snort/preproc_rules  
  
# These were commented out  
include $PREPROC_RULE_PATH/preprocessor.rules  
include $PREPROC_RULE_PATH/decoder.rules  
include $PREPROC_RULE_PATH/sensitive-data.rules  
  
# Listed as /usr/local/lib when they were actually in /usr/lib  
dynamicpreprocessor directory /usr/lib/snort_dynamicpreprocessor/  
dynamicengine /usr/lib/snort_dynamicengine/libsf_engine.so  
dynamicdetection directory /usr/lib/snort_dynamicrules  
  
# Increased to 65535 to avoid startup errors (I don't know the full reason yet)  
preprocessor http_inspect: global iis_unicode_map unicode.map 1252 compress_depth 65535 decompress_depth 65535  
  
# This was commented out; detects portscans!  
preprocessor sfportscan: proto  { all } memcap { 10000000 } sense_level { low }  
  
# MySQL connection params  
output database: alert, mysql, user=snort password=PASSWORD dbname=snort host=localhost  
output database: log, mysql, user=snort password=PASSWORD dbname=snort host=localhost
```

Then uncomment all the preprocessor rules includes.

### <font color="red">IMPORTANT</font>: Logging to MySQL

Although the Snort config above *looks* like it should log everything to
your MySQL database, you'll soon find that it doesn't. Running the test
command in the section below will lead to Snort declaring that
everything's peachy. It will create a single row in `snort.sensor` and
nothing else.

I wasted about 4 hours attempting to reconfigure Snort, reading through
the config file again, making sure my DB was okay. It was [this post on
the Snort
forum](https://forums.snort.org/forums/snort-newbies/topics/snort-not-logging-to-mysql-database#post_57266)
that made my day. You basically open up `/etc/sysconfig/snort` and
comment out this stupid line:

    ALERTMODE=fast

That's it. Restart `snortd` and Snort will now log to `/var/log/snort`
*and* your MySQL instance. Thanks "aline"!

### <font color="red">IMPORTANT</font>: Logging to custom logfiles

It took me 24 painstaking hours to figure this out. I can write a poem
on the frustration. Basically, Snort wouldn't write to a logfile of my
choice and I didn't know why. Turns out that I had to turn off binary
logs in `/etc/sysconfig/snort`:

    BINARY_LOG=0

If you don't do this, Snort won't care what you specify your unified2
logfiles to be; it will *always* write to `snort.log.<timestamp>`.

Test Run
--------

Check your configuration with this:

    /usr/sbin/snort -T -u snort -g snort -c /etc/snort/snort.conf

Using the `-D` flag will run snort in daemon mode and is not useful. If
everything looks OK,

    [root@snort snort]# service snortd start  
    Starting snort: Spawning daemon child...  
    My daemon child 30815 lives...  
    Daemon parent exiting                           [  OK  ]

Installing Oinkmaster
---------------------

This downloads and updates rulesets on a cron job. You'll need an
"Oinkcode" which you can procure from your Snort.org account.

    cd /opt  
    wget -O - http://prdownloads.sourceforge.net/oinkmaster/oinkmaster-2.0.tar.gz?download | tar -xvzf -  
    ln -s oinkmaster-2.0 oinkmaster  
    cd oinkmaster  
    cp oinkmaster.conf oinkmaster.conf.original  
    mkdir tmp backup.rules

Then edit the config file to specify URLs and oinkcodes:

    url = http://www.snort.org/pub-bin/oinkmaster.cgi/<your code>/snortrules-snapshot-2900.tar.gz  
    tmpdir = /opt/oinkmaster/tmp

I prefer keeping oinkmaster in `/opt`. For a trial run, I issue:

    mkdir /tmp/snort  
    /opt/oinkmaster/oinkmaster.pl -c -v -C /opt/oinkmaster/oinkmaster.conf -o /tmp/snort/

As a poor registered user, you have to wait 15 minutes between
downloads. If all's well, you can now set a `cron` job with:

    # Download new Snort rulesets every Friday at midnight  
    0 0 * * 5 /opt/oinkmaster/oinkmaster.pl -C /opt/oinkmaster/oinkmaster.conf \  
              -o /etc/snort/rules/ -b /opt/oinkmaster/backup.rules  \  
              2>&1 | mail -s "snrt - oinkmaster" support@example.com

Backs up rules, emails you everything on Friday at midnight. Awesome.

Using Barnyard to Spool Data
----------------------------

### Preconfiguring Snort

Open up `/etc/snort/snort.conf` change this line:

    output unified2: filename merged.log, limit 128, nostamp, mpls_event_types, vlan_event_types

to this:

    output unified2: filename merged.log, limit 128

For some reason, *barnyard2 will not work* if you don't do this. Now
comment out *every other* `output` declaration and restart Snort.

### Installing Barnyard

*   **Important**: You need to install
    [**barnyard2**](http://www.securixlive.com/barnyard2/) and *not*
    barnyard! The latter hasn't been updated in 4 years and is horribly
    difficult to set up. Barnyard2 also uses the newer "unified2" output
    format of Snort. Compilation will install everything in
    `/usr/local/{bin,etc}`. I've [archived the
    RPM](http://support.example.com/software/archive/barnyard2-1.9-1.i386.rpm)
    if you don't feel like it.
*   **Important**: Make sure that you've set up your MySQL database and
    that it's completely empty. I'd drop and re-create if unsure.
*   "Continuous mode" refers to barnyard continually processing a single
    logfile that's created when the Snort service is started. You can
    manually crunch through multiple log files in "batch" mode.

Edit `/usr/local/etc/barnyard2.conf`. Here's a diff of the original
(&lt;) and my modifications (&gt;):

    60,61c60,61  
    < #config hostname: thor  
    < #config interface:  eth0  
    ---  
    > config hostname:  snort.example.com  
    > config interface: eth0  
    115c115  
    < #config show_year  
    ---  
    > config show_year  
    131c131  
    < #config waldo_file: /tmp/waldo  
    ---  
    > config waldo_file: /var/log/snort/barnyard2.waldo  
    152c152  
    < #config archivedir: /tmp  
    ---  
    > config archivedir: /var/log/barnyard2  
    215c215  
    < output alert_fast: stdout  
    ---  
    > #output alert_fast: stdout  
    324,325c324,325  
    <   
    <   
    ---  
    > output database: log, mysql, user=snort password=PASSWORD dbname=snort host=localhost  
    > output database: alert, mysql, user=snort password=PASSWORD dbname=snort host=localhost

Now set up the waldo file and a few folders:

    mkdir /var/log/barnyard2  
    chmod 666 /var/log/barnyard2  
    chown snort:snort /var/log/barnyard2  
    touch /var/log/snort/barnyard2.waldo  
    chown snort:snort /var/log/snort/barnyard2.waldo

I highly recommend testing this configuration:

    /usr/local/bin/barnyard2 -T -c /usr/local/etc/barnyard2.conf

Now start barnyard in daemon mode:

    /usr/local/bin/barnyard2 -c /usr/local/etc/barnyard2.conf \  
                             -d /var/log/snort \  
                             -f merged.log \  
                             -w /var/log/snort/barnyard2.waldo \  
                             -a /var/log/barnyard2 \  
                             -u snort -g snort \  
                             -D

You can now add this snippet to `/etc/rc.local` run barnyard2 at
startup.

Installing BASE
---------------

First make sure you have `php-adodb` installed (available at EPEL). This
installs everything in `/usr/share/php/adodb`.

    yum -y install php-adodb --enablerepo=epel

Then install BASE in your siteroot (assuming default here):

    cd /var/www/html/  
    wget -O http://downloads.sourceforge.net/project/secureideas/BASE/base-1.4.5/base-1.4.5.tar.gz | tar -xvzf -  
    ln -s base-1.4.5 base  
    cp base/base_conf.php.dist base_conf.php

Then edit the `base_conf.php` file. Here are my additions/modifications
(other than the Snort MySQL and SMTP info that BASE needs). I used the
same MySQL settings for the "Archive DB" parameters.

    $BASE_urlpath = 'http://snort.example.com/base';  
    $DBlib_path = '/usr/share/php/adodb';  
    $resolve_IP = 1;

Now navigate to the page and click "Create Base AG". BAM! Now make sure
that you at least `htpasswd` the page, since BASE doesn't have any login
system.

Sources
-------

*   [Installing Snort and Base on CentOS 5.2](http://www.how-to-linux.com/centos-52/install-snort-and-base-on-centos-52/)
*   [Install Snort 2.8.6 on CentOS 5.5 (Official)](http://www.snort.org/assets/145/Install_Snort_2.8.6_on_CentOS_5.5.pdf)
*   [Super-detailed installation log on Fedora](http://www.rootninja.com/snort-ids-basic-analysis-security-engine-base-fedora/)
*   [Installing Snort Report](http://www.symmetrixtech.com/articles/008-snortinstallguide290.pdf)
*   [Presentation on Snort and Barnyard](Media:Barnyard_Presentation.ppt "wikilink")

Miscellaneous
-------------

### Barnyard2 compilation flags

    ./configure --with-mysql=/usr/bin/mysql \  
                --with-mysql-includes=/usr/include/ \  
                --with-mysql-libraries=/usr/lib/mysql/

### A Note on Waldo Files

To keep track of the most recent logfiles between service restarts,
barnyard uses a "waldo" file. Its contents look like this:

    /var/log/snort snort.log 1305050676 0

`1305050676` is the most recent timestamp. *However*, with the setup
above (where I just `touch`-ed the file), the contents generated by my
command look like this:

    /var/log/snort  
      
      
      
    merged.log  
      
      
      
    ??M?

Strange.
