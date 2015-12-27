This is a quick overview of `logrotate` which... umm... rotates log
files. This was written for CentOS/RHEL 5.5.

Pertinent Files & Directories
-----------------------------

|                    File/Dir                   |                                                                  Purpose                                                                   |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| `/usr/sbin/logrotate`                         | The command itself.                                                                                                                        |
| `/etc/cron.daily/logrotate`                   | Bash script that executes the `logrotate` command every day.                                                                               |
| `/etc/logrotate.conf` and `/etc/logrotate.d/` | Rotation defaults if they are not defined for specific daemons. You can add rotate configs to this file, or put them in `/etc/logrotate.d` |
| `/var/lib/logrotate.status`                   | Show when the specified log files were last rotated.                                                                                       |

Logrotate Options
-----------------

Quite simple, really. The `man` page elucidates all available options.
Here's a quick table of what the most commonly used ones do.

|              Option             |                                                              Purpose                                                               |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| `rotate <number>`               | Keep specified number of logfiles before they are deleted and/or emailed to admin                                                  |
| `size=<number>`                 | Rotate logs when they reach this size.                                                                                             |
|                                 | This is done ''without regard'' for the last rotated time.                                                                         |
|                                 | Use `minsize <number>` if you'd like to balance both time and size.                                                                |
|                                 | Bytes are used without a specifier. 100k, 100M are also fine.                                                                      |
| `daily`                         | Rotate daily. This is the default minimum granularity, unless you move `/etc/cron.daily/logrotate` to `/etc/cron.hourly/logrotate` |
| `weekly`                        | Rotate weekly                                                                                                                      |
| `monthly`                       | Rotate monthly                                                                                                                     |
| `notifempty`                    | Don't rotate if empty (the reverse, `ifempty`, is default)                                                                         |
| `missingok`                     | If log file is missing for some reason, move on without error                                                                      |
| `compress`                      | Compress logfile after rotation.                                                                                                   |
|                                 | Default is `gzip` with `-9` (maximum) compression.                                                                                 |
|                                 | The choice of compression program can be changed with `compresscmd` (e.g. `compresscmd bzip2`).                                    |
|                                 | To pass options (like compression level), use `compressoptions`.                                                                   |
|                                 | To change extension of compressed file, use `compressext`                                                                          |
| `dateext`                       | Use YYYYMMDD format instead of just tacking on numbers (like `.0`, `.1` and so on) to the rotated files.                           |
| `mail <recipient>`              | Email recipient the file that will be deleted after a rotation cycle                                                               |
| `create <mode> <owner> <group>` | Create logfiles with the specified permissions and owner:group attributes.                                                         |
| `olddir <directory>`            | Move all but the newest log file to this directory (nice to keep things organized)                                                 |

### Other (kinda important) options

You'll see these used with Apache, for instance.

|      Option     |                                                           Purpose                                                            |
|-----------------|------------------------------------------------------------------------------------------------------------------------------|
| `prerotate`     | Execute the a script ''before'' rotating a log. Should end this with `endscript`.                                            |
| `postrotate`    | Execute the a script ''after'' rotating a log. Should end this with `endscript`.                                             |
| `sharedscripts` | Make sure that the script(s) specified in `prerotate` and/or `postrotate` run just ''once'' (i.e. not for ''every'' logfile) |
| `delaycompress` | Don't compress yesterday's logfile (if daily... you get the picture)                                                         |

Example
-------

Here's a real-world application of the above. I want to rotate an
already huge log of OpenVPN connections (\~32M in size) like so:

*   I want to maintain **60 days** worth of logs
*   Logfile **size doesn't matter** to me
*   The files should be rotated in **YYYYMMDD** format
*   They should be **compressed** (the default `gzip -9` is fine)
*   They should be **organized**; older logfiles should be **in a
    separate directory**

The logfile is at `/var/log/openvpn/connections.log`. Here's the
configuration file I created for the above. It's `/etc/logrotate.d/openvpn`:

    /var/log/openvpn/connections.log {  
        daily  
        rotate 60  
        dateext  
        compress  
        olddir /var/log/openvpn/old  
      
        nomail  
        missingok  
        notifempty  
        delaycompress  
        create 640 root root  
    }

To test this, I run:

    logrotate -d /etc/logrotate.d/openvpn

The `-d` switch runs it in verbose, debug, dry-run mode (i.e. nothing
actually happens.)

Other Stuff
-----------

### Leopard and Snow Leopard

These use a new utility called `newsyslog`. It is invoked every minute
by:

    /System/Library/LaunchDaemons/com.apple.newsyslog.plist

The config file for this is `/etc/newsyslog.conf` and is kinda nicer
than `logrotate`'s config :)

Sources
-------

-   [Logrotate Examples](http://www.thegeekstuff.com/2010/07/logrotate-examples/)
-   [Using logrotate to manage log files](http://linuxers.org/howto/howto-use-logrotate-manage-log-files)
-   [Rotating Linux log files](http://www.ducea.com/2006/06/06/rotating-linux-log-files-part-2-logrotate/)
