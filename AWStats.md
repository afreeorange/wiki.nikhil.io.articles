Pre-Flight
----------

-   [AWStats](http://awstats.sourceforge.net/) 7.2 installed on a CentOS
    6.3 box
-   Nginx will serve the generated statistics
-   Trying to set up analytics for `blog.example.com`
-   Log files at `/var/log/nginx`
    -   Logrotated and compressed every day
-   Stats site will be at `/var/www/html/stats`
-   AWStats data directory will be at `/var/lib/awstats`

Installation
------------

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

Don't forget to make it executable and try it out :)

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

Other notes
-----------

-   `awstats_updateall.pl` is just a wrapper for `awstats.pl` that runs
    all found configurations in `/etc/awstats`.
-   I couldn't find a way to get the select dropdown and sidebar to be
    generated statically.

References
----------

-   [This
    guy](http://kamisama.me/2013/03/20/install-configure-and-protect-awstats-for-multiple-nginx-vhost-on-debian/)
    runs AWStats as a CGI application.

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
[Category: Linux](Category:_Linux "wikilink")
