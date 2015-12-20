Installation
------------

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

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
[Category: Linux](Category:_Linux "wikilink")
