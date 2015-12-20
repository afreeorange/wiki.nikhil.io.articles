Pre-Flight
----------

-   [AWStats](http://awstats.sourceforge.net/) 7.2 installed on a CentOS
    6.3 box
-   Nginx will serve the analytics statically (i.e., [*no*
    CGI](http://winterdrake.com/awstats-tip-creating-static-pages/).)
-   Trying to set up analytics for `blog.example.com`
-   Server log files at `/var/log/nginx`
    -   Logrotated and compressed every day
-   Stats site will be at `/var/www/html/stats`
-   AWStats data directory will be at `/var/lib/awstats`

Installation
------------

-   Downloaded and installed RPM [from
    website](http://awstats.sourceforge.net/).
-   Installs in `/usr/local/awstats`.
-   You'll find the model config in `/etc/awstats/awstats.model.conf`

`   mv /etc/awstats/{awstats.model.conf,awstats.conf}`

Since I'm setting up analytics for a blog at `http://blog.example.com`,

`   cp /etc/awstats/{awstats.conf,awstats.`**`blog.example.com`**`.conf}`  
`   mkdir -p /var/lib/awstats/`**`blog.example.com`**

Then modified these params

`   LogFile="/var/log/nginx/blog.access.log"`  
`   SiteDomain="blog.example.com"`  
`   HostAliases="blog.example.com localhost 127.0.0.1"`  
`   DNSLookup=1`  
`   DirData="/var/lib/awstats/blog.example.com"`  
`   EnableLockForUpdate=1`

### Cron entry

This generates the static HTML pages. Since my logrotate config runs
daily, I'll set the job to run at that frequency as well.

I added this to `/etc/cron.daily/awstats-blog`:

    #!/bin/bash
    STATIC_DIR="/var/www/html/stats"

    YEAR=$(date +"%Y")
    MONTH=$(date +"%m")
    LOG_DIR=$STATIC_DIR/$YEAR/$MONTH

    mkdir -p $LOG_DIR
    /usr/local/awstats/tools/awstats_buildstaticpages.pl -dir=$LOG_DIR -config=blog.example.com -update

Don't forget to make it executable.

### Generate Data Files

`   /usr/local/awstats/tools/awstats_updateall.pl now`

Scan for any errors, fix accordingly. You can now see a text file (the
'database' file) in `/var/lib/awstats/blog.example.com`

### Generate Static HTML

Simply run your cron script

`   /etc/cron.daily/awstats-blog`

### Nginx

First, define where the static HTML analytics files are:

`   location /stats {`  
`       root /var/www/html;`  
`       autoindex on;`  
`   }`

Now some symlink gymnastics

`   ln -s /usr/local/awstats/wwwroot /usr/local/awstats/stats`

Now add a definition for the icons:

`   location /stats/icon {`  
`       root /usr/local/awstats;`  
`   }`

### The logrotate issue

I configured logrotate to compress my logfiles. This can be problematic,
but has a simple solution: tell AWStats to update its data files
*before* logrotate does anything with them.

So,

`   /var/log/nginx/*.log {`  
`       daily`  
`       missingok`  
`       rotate 52`  
`       compress`  
`       delaycompress`  
`       notifempty`  
`       create 640 nginx adm`  
`       sharedscripts`  
`       `**`prerotate`**  
`           `**`/usr/local/awstats/tools/awstats_updateall.pl` `now`**  
`       `**`endscript`**  
`       postrotate`  
``            [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid` ``  
`       endscript`  
`   }`

Importing historic log data
---------------------------

### Generating data files

I had previous logfiles in `/var/log/nginx` that looked like this as a
result of logrotate:

`   blog.access.log-20130625.gz`  
`   blog.access.log-20130626.gz`  
`   blog.access.log-20130628.gz`  
`   blog.access.log-20130701.gz`  
`   blog.access.log-20130703.gz`  
`   blog.access.log-20130704.gz`  
`   blog.access.log-20130705.gz`

To import these, I removed the AWStats database files in
`/var/lib/awstats/blog`. I then temporarily changed the "`LogFile`"
parameter in the config file (`/etc/awstats/awstats.blog.example.conf`)
to this:

`   LogFile="zcat /var/log/nginx/blog.access.log*.gz |"`

Should be self-exlanatory. Then ran the update script as usual:

`   /usr/local/awstats/tools/awstats_updateall.pl now`

This generated the older database entries. There are
[other](http://shebangme.blogspot.com/2012/04/awstats-rebuilding-files-from-old-logs.html)
[methods](http://adamish.com/blog/archives/251) as well, especially if
you don't have access to your older records.

You should now regenerate the static HTML.

### Regenerating static HTML pages

For the months and years you have log files for, write two small `for`
loops!

    for YEAR in $(seq 2010 2013); do
        for MONTH in $(seq --format="%02g" 5 8); do
            STATSDIR=/var/www/html/stats/$YEAR/$MONTH
            mkdir -p $STATSDIR
            /usr/local/awstats/tools/awstats_buildstaticpages.pl -dir=$STATSDIR -month=$MONTH -year=$YEAR -config=blog.example.com
        done
    done

Ta da!

Plugins
-------

### GeoIP

Will be *much* faster than DNS. A little painful, but worth it

#### Perl Module

You'll need the C API first.

`   # Get the latest source (1.5+)`  
`   wget -O - `[`http://www.maxmind.com/download/geoip/api/c/GeoIP-latest.tar.gz`](http://www.maxmind.com/download/geoip/api/c/GeoIP-latest.tar.gz)` | tar -xvzf -`  
`   cd GeoIP-1.5.1`  
`   ./configure; make; make install`

`   # Make sure CPAN can find the compiled libs`  
`   echo "/usr/local/lib" > /etc/ld.so.conf.d/GeoIP.conf`  
`   /sbin/ldconfig /etc/ld.so.conf -v`

You *should* be able to install this now via CPAN, but the module's
`Makefile` is screwed up (at least as of version 1.42). So download
directly and compile:

`   wget -O - `[`http://search.cpan.org/CPAN/authors/id/B/BO/BORISZ/Geo-IP-1.42.tar.gz`](http://search.cpan.org/CPAN/authors/id/B/BO/BORISZ/Geo-IP-1.42.tar.gz)` | tar -xzvf -`  
`   cd Geo-IP-1.42`  
`   perl Makefile.PL LIBS="-L/usr/local/lib -lGeoIP" INC=-I/usr/local/include`  
`   make`  
`   make install`

#### Database File

`   mkdir /opt/GeoIP`  
`   wget -O - `[`http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz`](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz)` | gunzip - > /opt/GeoIP/data`

Now uncomment this in your site config (also uncomment `DNSLookup`):

`   LoadPlugin="geoip GEOIP_STANDARD /opt/GeoIP/data"`

Run the update script!

### IPv6 Support

`   # Make sure you have CPAN first`  
`   yum -y install perl-CPAN`

`   # Open a prompt`  
`   cpan`

`   # Now type:`  
`   cpan[1]> install Net::IP`  
`   cpan[2]> install Net::DNS`

Uncomment `LoadPlugin="ipv6"`.

### Others

`   LoadPlugin="graphgooglechartapi"`  
`   LoadPlugin="hostinfo" # You'll need to install Net::XWhois via CPAN for this`  
`   LoadPlugin="tooltips"`

Other notes
-----------

-   `awstats_updateall.pl` is just a wrapper for `awstats.pl` that runs
    all found configurations in `/etc/awstats`.
-   I couldn't find a way to get the select dropdown and sidebar to be
    generated statically.
-   If you wanted to host on a subdomain, here's a basic Nginx config
    with some basic HTTP auth.

`   server {`  
`       listen 80;`  
`       server_name stats.example.com;`  
` `  
`       location / {`  
`           root /var/www/html/stats;`  
`           autoindex on;`  
`           `  
`           # Password-protect (something's better than nothing)`  
`           auth_basic            "Restricted";`  
`           auth_basic_user_file  /etc/nginx/conf.d/passwords;`  
`       }`  
` `  
`       location /icon {`  
`           root /usr/local/awstats/wwwroot;`  
`       }`  
`   }`

The `passwords` file is generated with the `htpasswd` command:

`   htpasswd -c /etc/nginx/conf.d/passwords username`

References
----------

-   [This
    guy](http://kamisama.me/2013/03/20/install-configure-and-protect-awstats-for-multiple-nginx-vhost-on-debian/)
    runs AWStats as a CGI application.

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
[Category: Linux](Category:_Linux "wikilink")
