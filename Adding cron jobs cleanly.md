Background
----------

Adding jobs to /etc/cron.x keeps things modular and 'clean'. You can add
hourly, daily, weekly and monthly (regular) scripts by creating your
script in the appropriate folder:
`/etc/cron.{hourly,daily,weekly,monthly}`

The cron daemon will run these hourly, daily, etc scripts at intervals
defined in `/etc/crontab`

` SHELL=/bin/sh`  
` PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`  
` `  
` # m h dom mon dow user    command`  
` 17 *  * * *   root    cd / && run-parts --report /etc/cron.hourly`  
` 25 6  * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )`  
` 47 6  * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )`  
` 52 6  1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )`

Observe the `PATH` variable set. This is why you don't need to provide
explicit paths (e.g. `/bin/rm`), but it's a good idea to do it anyway.

If you need to run a script akin to adding it to the crontab, you'll
need to place it in `/etc/cron.d`

Adding regular scripts
----------------------

Let's say you want something to run weekly.

-   Create your script
-   Move it to `/etc/cron.weekly`

Before you rejoice,

-   You *must* add the `#!/bin/sh` line to the top of your scripts! Not
    doing so will result in `Exec format error` messages.
-   You *must* set the permissions to 755

Adding crontab-style scripts
----------------------------

You'd add these to `/etc/cron.d/''scriptname''`. The format's the same,
except that you must *explicitly* provide the user executing the script
(makes sense.) For example:

` #!/bin/sh`  
` 09,39 * * * *  root  /bin/rm -rf /tmp/junk/*`

Add that to a file, move it to `/etc/cron.d`. Make sure that:

-   The `#!/bin/sh` line exists and is first.
-   The permissions to 755.

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
