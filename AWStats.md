-   Installation on a CentOS 7 box with PHP-FPM and Nginx from the
    base repo.
-   Statistics will be for `example.com`, served by Nginx.
-   Will install AWStats on a sub-domain, `statistics.example.com`
    -   Sub-domain root is `/var/www/html/statistics`
    -   Statically generated pages will be in
        `/var/www/html/statistics/pages`

How it works!
-------------   

AWStats is a Perl program that parses any log files you throw at it,
then creates a text-based 'database' (in `/var/lib/awstats` in this
guide).

Once generated/updates, statistics in this database can be viewed:

-   Dynamically with a CGI script (`awstats.pl`)
-   Via statically generated HTML files (`awstats_buildstaticpages.pl`)

AWStats is pretty ancient and can do a *lot* more, but as far as
installation's concered, that's all you should know.

Pre-Flight
----------

```bash
# Create requisite folders
mkdir /usr/local/awstats /var/lib/awstats /var/www/html/statistics

# Set up AWStats
cd && wget -O - http://www.awstats.org/files/awstats-7.4.tar.gz | tar -xvzf -
mv ~/awstats-7.4/* /usr/local/awstats/

# Set appropriate permissions. PHP-FPM runs as apache
chown -R apache:apache /usr/local/awstats

# Now run the configuration script
perl /usr/local/awstats/tools/awstats_configure.pl
```

[Here's the transcript](:File:awstats-install-transcript.txt "wikilink"). All I
needed from it was a sample config file for the `example.com` domain.

Set up a Configuration
----------------------

Modify `/etc/awstats/awstats.example.com.conf` to edit the path to the
Nginx access log file.

    LogFile="/var/log/nginx/example.com.access.log"

Modify other parameters later.

Populate the AWStats 'Database'
-------------------------------

    perl /usr/local/awstats/wwwroot/cgi-bin/awstats.pl -update -config=example.com

### Automation

This is done with:

    /usr/local/awstats/tools/awstats_updateall.pl now

which you should add to `/etc/logrotate.d/nginx` as a pre-rotation
script

    /var/log/nginx/*.log {
            daily
            missingok
            rotate 52
            compress
            delaycompress
            notifempty
            create 640 nginx adm
            sharedscripts
            prerotate
                /usr/local/awstats/tools/awstats_updateall.pl now
            endscript
            postrotate
                [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
            endscript
    }

Stats will be updated the next time any of those log files are rotated.
You can also add a `cron` entry:

    0 * * * * /usr/local/awstats/tools/awstats_updateall.pl now > /dev/null 2>&1

Set up Static Pages
-------------------

You don't have to set up *both* the CGI-BIN viewer and static pages BTW.
You could simply generate static pages and not allow any FastCGI
execution.

A single script generates static pages:

    /usr/local/awstats/tools/awstats_buildstaticpages.pl \  
                    -update \  
                    -config=example.com \  
                    -dir=/var/www/html/statistics.example.com/pages \  
                    -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl

Try it out and you'll see a bunch of HTML files in
`/var/www/html/statistics.example.com/pages`

### Automation

Same as with automating the database script: add to
`/etc/logrotate.d/nginx`:

    /var/log/nginx/*.log {
            daily
            missingok
            rotate 52
            compress
            delaycompress
            notifempty
            create 640 nginx adm
            sharedscripts
            prerotate
                /usr/local/awstats/tools/awstats_updateall.pl now
                /usr/local/awstats/tools/awstats_buildstaticpages.pl -update -config=example.com -dir=/var/www/html/statistics.example.com/pages -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl
            endscript
            postrotate
                [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
            endscript
    }

Set up Nginx
------------

AWStats 7.4 ships with a PHP wrapper that Nginx can hand to a FastCGI
process (PHP-FPM in this case).

### Configuration

    # statistics.example.com
    server {
        listen      0.0.0.0:80;
        server_name statistics.example.com;

        access_log off;
        error_log  off;

        location / {
            root  /var/www/html/statistics.example.com;
        }

        location /classes/ {
            alias /usr/local/awstats/wwwroot/classes/;
        }

        location /css/ {
            alias /usr/local/awstats/wwwroot/css/;
        }

        location /icon/ {
            alias /usr/local/awstats/wwwroot/icon/;
        }

        location /js/ {
            alias /usr/local/awstats/wwwroot/js/;
        }

        # Dynamic stats
        location ~ ^/cgi-bin/(awredir|awstats)\.pl {
            gzip off;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_param SCRIPT_FILENAME   /usr/local/awstats/tools/nginx/awstats-fcgi.php;
            fastcgi_param X_SCRIPT_FILENAME /usr/local/awstats/wwwroot$fastcgi_script_name;
            fastcgi_param X_SCRIPT_NAME $fastcgi_script_name;
            include fastcgi_params;
        }
    }

### Protect Pages

Enable SSL (even with a shitty self-signed certificate), listen on port
443, then add this to your `server` definition:

    auth_basic            "Restricted";
    auth_basic_user_file  passwords;

And here's how you could generate passwords without the full suite of
Apache tools:

    PASSWORD="ThePassword";
    SALT="$(openssl rand -base64 3)";
    SHA1=$(printf "$PASSWORD$SALT" | openssl dgst -binary -sha1 | sed 's#$#'"$SALT"'#' | base64);
    printf "the_user:{SSHA}$SHA1\n"

Arch Linux Notes
----------------

### Installation

    pacman -Syu awstats

### Configuration

    perl /usr/share/awstats/tools/awstats_configure.pl

Stuff ends up in `/etc/awstats`

### Logrotate config

    # /etc/logrotate.d/nginx
    /var/log/nginx/*log {
        compress
        create 640 http log
        missingok
        sharedscripts
        su http log
        daily
        delaycompress
        notifempty
        rotate 52
        prerotate
            perl /usr/share/awstats/tools/awstats_updateall.pl -awstatsprog=/usr/share/webapps/awstats/cgi-bin/awstats.pl now
        endscript
        postrotate
            test ! -r /run/nginx.pid || kill -USR1 `cat /run/nginx.pid`
        endscript
    }
