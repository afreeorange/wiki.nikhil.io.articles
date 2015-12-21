Mostly for RHEL5 systems. Should work on any Linux distro.

Preliminary notes
-----------------

The config for `ntpd` is `/etc/ntp.conf`. These lines are of interest:

    server 0.centos.pool.ntp.org  
    server 1.centos.pool.ntp.org  
    server 2.centos.pool.ntp.org

They must be changed to whatever you have in mind for NTP servers.
**Important**: You *must* also include these servers in the
`/etc/ntp/step-tickers` file! This file will be found initially empty.
If you don't, you'll see something like this when restarting the NTP
daemon:

    [root@localhost ntp]# /sbin/service ntpd start
    ntpd: Synchronizing with time server:            [FAILED]
    Starting ntpd:                                   [  OK  ]

The NTP port is **123** and must be open in your iptables rules to allow
synchronization.

The University of Iowa NTP servers
----------------------------------

As mentioned earlier, these need to be in the `step-tickers` file.

` ntp1.uiowa.edu`  
` ntp2.uiowa.edu`  
` ntp3.uiowa.edu`

How Time Zones work in Linux
----------------------------

**Important**: The NTP servers do *not* provide timezone-specific data!
All they provide is UTC time. This data is then adjusted by `ntpd` in
conjunction with `/etc/localtime`.

The `/etc/localtime` file has a list of the zone offsets (going till
2038; the 32-bit problem) and, importantly, the dates on which the clock
is moved forward and backward (for the Daylight Savings crap.) To see
the contents of this file, you have to use `zdump`:

` zdump -v /etc/localtime | grep 2010`

This will show how the clock is adjusted w.r.t. timezone and DST:

`/etc/localtime  Sun Mar 14 07:59:59 2010 UTC = Sun Mar 14 01:59:59 2010 CST isdst=0 gmtoff=-21600`  
`/etc/localtime  Sun Mar 14 08:00:00 2010 UTC = Sun Mar 14 03:00:00 2010 CDT isdst=1 gmtoff=-18000`  
`/etc/localtime  Sun Nov  7 06:59:59 2010 UTC = Sun Nov  7 01:59:59 2010 CDT isdst=1 gmtoff=-18000`  
`/etc/localtime  Sun Nov  7 07:00:00 2010 UTC = Sun Nov  7 01:00:00 2010 CST isdst=0 gmtoff=-21600`

Comprehensive timezone resources are located in `/usr/share/zoneinfo`.
It's the directory utilized by, for example, Anaconda to set your
timezone at install. Here's a sample tree view of the contents of this
folder.

` America`  
` |-- Anguilla`  
` |-- Antigua`  
` |-- Araguaina`  
` |-- Argentina`  
` |   |-- Buenos_Aires`  
` |   |-- Catamarca`  
` |   |-- ComodRivadavia`  
` |   |-- Cordoba`  
` |   |-- Jujuy`  
` |   |-- La_Rioja`  
` |   |-- Mendoza`  
` |   |-- Rio_Gallegos`  
` |   |-- Salta`  
` |   |-- San_Juan`  
` |   |-- San_Luis`  
` |   |-- Tucuman`  
``  |   `-- Ushuaia ``  
` |-- Aruba`  
` |-- Asuncion`  
` |-- Atikokan`  
` .`  
` .`  
` .`

To manually adjust your timezone, you just need to symlink the correct
resource file to `/etc/localtime`.

For US zones, the naming scheme is slightly weird. Here's a table of
files for four US timezones. They are at the root of
`/usr/share/zoneinfo`

| Time Zone | File in /usr/share/zoneinfo |---- | Eastern | EST5EDT |---- | Central | CST6CDT |---- | Mountain | MST7MDT |---- | Pacific | PST8PDT |---- |
|-----------|-----------------------------------|---------|---------------|---------|---------------|----------|---------------|---------|---------------|

Again, use `zdump` to view/verify these files.

Adjusting Time Zones
--------------------

### The simple way

You just have to symlink the correct time resource file to
`/etc/localtime`. For instance, if I'm in the America/Chicago timezone:

` mv /etc/localtime /etc/localtime.backup`  
` ln -s /usr/share/zoneinfo/America/Chicago /etc/localtime`  
` service ntpd restart`

### tzselect

If you don't want to bother with pesky symlinking to update
`/etc/localtime`, you can always use `tzselect`. It's a small,
interactive command-line utility which will generate a string like this
that you can add to `~/.bash_profile`

` TZ='Africa/Porto-Novo'; export TZ`

Fixing Daylight Savings Issues
------------------------------

A `zdump` of the `/etc/localtime` file will yield what the local system
does at certain dates.

` zdump -v /etc/localtime`

Look at the output. If you see the correct dates for Daylight Savings,
you're good. For example,

` /etc/localtime  Sun Mar 14 07:59:59 2010 UTC = Sun Mar 14 01:59:59 2010 CST isdst=0 gmtoff=-21600`  
` /etc/localtime  Sun Mar 14 08:00:00 2010 UTC = Sun Mar 14 03:00:00 2010 CDT isdst=1 gmtoff=-18000`  
` /etc/localtime  Sun Nov  7 06:59:59 2010 UTC = Sun Nov  7 01:59:59 2010 CDT isdst=1 gmtoff=-18000`  
` /etc/localtime  Sun Nov  7 07:00:00 2010 UTC = Sun Nov  7 01:00:00 2010 CST isdst=0 gmtoff=-21600`

Which seems correct, since the time jumped ahead an hour on March 14.
But here's some worrisome output:

` /etc/localtime  Sun `**`Apr`
`4`**` 07:59:59 2010 UTC = Sun Apr  4 01:59:59 2010 CST isdst=0 gmtoff=-21600`  
` /etc/localtime  Sun `**`Apr`
`4`**` 08:00:00 2010 UTC = Sun Apr  4 03:00:00 2010 CDT isdst=1 gmtoff=-18000`  
` /etc/localtime  Sun `**`Oct`
`31`**` 06:59:59 2010 UTC = Sun Oct 31 01:59:59 2010 CDT isdst=1 gmtoff=-18000`  
` /etc/localtime  Sun `**`Oct`
`31`**` 07:00:00 2010 UTC = Sun Oct 31 01:00:00 2010 CST isdst=0 gmtoff=-21600`

In this case, you'll need to update the timezone files in
`/usr/share/zoneinfo`.

**Note:** The timezone files are all compiled with `'''zic'''`:
<http://www.gsp.com/cgi-bin/man.cgi?section=8&topic=zic>

### The easy way - Use YUM

One way to do it is to update the `tzdata` RPM.

` yum update tzdata`

This should fetch and install the correct files.

### The hard way - Compile with Zic

Download the latest data. This is the file as of 2010:

` wget '`[`ftp://elsie.nci.nih.gov/pub/tzdata2010f.tar.gz`](ftp://elsie.nci.nih.gov/pub/tzdata2010f.tar.gz)`'`

They don't extract to a directory, so make sure you create one! Now
compile zone files using `zic` with this syntax:

` zic -d `<directory to extract to>` `<name of zone file>

Let's try it for North America. The file to compile is called
`northamerica`. Let's compile the results to `tempdir`

` zic -d tempdir northamerica`

Now verify that the newly compiled files have the correct DST offsets:

` [root@localhost tempdir]# zdump -v tempdir/CST6CDT | grep 2010`  
` CST6CDT  Sun Mar 14 07:59:59 2010 UTC = Sun Mar 14 01:59:59 2010 CST isdst=0 gmtoff=-21600`  
` CST6CDT  Sun Mar 14 08:00:00 2010 UTC = Sun Mar 14 03:00:00 2010 CDT isdst=1 gmtoff=-18000`  
` CST6CDT  Sun Nov  7 06:59:59 2010 UTC = Sun Nov  7 01:59:59 2010 CDT isdst=1 gmtoff=-18000`  
` CST6CDT  Sun Nov  7 07:00:00 2010 UTC = Sun Nov  7 01:00:00 2010 CST isdst=0 gmtoff=-21600`

Woohoo! Now you can copy these over to `/usr/share/zoneinfo`:

` [root@localhost tempdir]# cp -r * /usr/share/zoneinfo/`

Now remove the old `/etc/localtime` and symlink the newly compiled zone
file:

` [root@localhost tempdir]# rm -f /etc/localtime`  
` [root@localhost tempdir]# ln -s /usr/share/zoneinfo/CST6CDT /etc/localtime`

Restart `ntpd` and you're good to go!

Resources
---------

1.  [Update Linux and FreeBSD systems for new Daylight Saving Time
    setting](http://articles.techrepublic.com.com/5100-10878_11-6163042.html)
2.  [Date, Time and Time Zones for Red Hat
    Linux](http://www.vanemery.com/Linux/RH-Linux-Time.html) - Excellent
    overview of NTP and date and time utilities.



