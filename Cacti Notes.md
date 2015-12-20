Pre-Flight
----------

I'm using CentOS-Testing to install PHP 5.2.10.

` yum install mysql-server mysql php-mysql php-pear php-common php-gd php-devel php php-mbstring php-cli php-snmp php-pear-Net-SMTP php-mysql httpd net-snmp-utils php-snmp net-snmp-libs rrdtool --enablerepo=c5-testing`

Here's what I have in `/etc/snmp/snmp/conf`:

` rocommunity HOME`  
` syslocation "The University of Iowa"`  
` syscontact mail@nikhil.io`

I also made sure that the SNMP daemon was running by running:

` snmpwalk -v 1 -c CLCG localhost IP-MIB::ipAdEntIfIndex`

Set up the MySQL database:

` grant all on cacti.* to 'cactiuser'@'localhost' identified by 'Lua77hQhdnB';`

Install Cacti
-------------

Make sure that [the EPEL
repository](http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-4.noarch.rpm)
is installed.

` yum -y install cacti --enablerepo=epel`

Look for an SQL install file and install it:

` rpm -ql cacti | grep cacti.sql`  
` /usr/share/doc/cacti-0.8.7f/cacti.sql`  
` mysql -ucactiuser -p cacti < /usr/share/doc/cacti-0.8.7f/cacti.sql`

Now edit the config file (`/etc/cacti/db.php`) to add database
information.

-   The Cacti RPM also installs a configuration directive,
    `/etc/httpd/conf.d/cacti.conf`. Change it to `Allow from all`
-   You'll also find stuff in `/etc/cron.d/cacti`. Add that to
    your crontab.
-   The installer should create a `cacti` user and `cacti` group.

Change some permissions:

` chmod -R 755 /usr/share/cacti`  
` chown -R cacti:cacti /usr/share/cacti`

Make sure that the `rra/` and `log/` are owned by `cacti`.

Now restart the `httpd` service and navigate to
`http://site/'''cacti'''` (or whatever you edited to be the `Alias`
directive.) The username/password config for the first time login is
admin/admin.

Problems
--------

-   **The most important thing**: The poller must run as the `cacti`
    user!

### You get a "Forbidden" when loading the page for the first time

-   Make sure that you have `Allow from all` in
    `/etc/httpd/conf.d/cacti.conf`
-   Make sure that the user:group permissions are good for
    `/usr/share/cacti`

### No freakin' graphs

Tailing the `httpd` error log produces something like:

` ERROR: opening '/usr/share/cacti/rra/localhost_mem_buffers_3.rrd': No such file or directory`

There's also nothing in `/var/log/cacti/cacti.log`. In this case, do
this:

` chown cacti:apache /var/lib/cacti/rra`  
` chmod 775 cacti:apache /var/lib/cacti/rra`

This is [a bug](https://bugzilla.redhat.com/show_bug.cgi?id=250348) that
others have encountered. I changed the permissions like above, ran the
poller and the graphs were displayed. I'm fine with this.

[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
