`incron` is an [inotify](http://inotify.aiken.cz/)-based cron system.
You can make it do stuff (like trigger a script) when files or
directories change, are opened, closed, etc (i.e., "filesystem events").
You need a 2.6.13 kernel or higher to use it.

Installation
------------

`# CentOS/RHEL`  
`sudo yum install incron --yes`  
  
`# Ubuntu`  
`sudo apt-get install incron --yes`

The global config file is `/etc/incron.conf`. Now make sure you start
the service (might be a good idea to `chkconfig` this too):

`# CentOS/RHEL`  
`service incrond start`  
  
`# Ubuntu`  
`/etc/init.d/incron start`

Events
------

You can get a listing of events by issuing:

`[root@example ~]# incrontab `**`-t`**  
`IN_ACCESS,IN_MODIFY,IN_ATTRIB,IN_CLOSE_WRITE,IN_CLOSE_NOWRITE,IN_OPEN,`  
`IN_MOVED_FROM,IN_MOVED_TO,IN_CREATE,IN_DELETE,IN_DELETE_SELF,IN_CLOSE,`  
`IN_MOVE,IN_ONESHOT,IN_ALL_EVENTS,IN_DONT_FOLLOW,IN_ONLYDIR,IN_MOVE_SELF`

You can find an explanation of each in `/usr/include/linux/inotify.h`.
For example:

`26 /* the following are legal, implemented events that user-space can watch for */`  
`27 #define IN_ACCESS               0x00000001      /* File was accessed */`  
`28 #define IN_MODIFY               0x00000002      /* File was modified */`  
`29 #define IN_ATTRIB               0x00000004      /* Metadata changed */`  
`30 #define IN_CLOSE_WRITE          0x00000008      /* Writtable file was closed */`  
`31 #define IN_CLOSE_NOWRITE        0x00000010      /* Unwrittable file closed */`  
`32 #define IN_OPEN                 0x00000020      /* File was opened */`  
`33 #define IN_MOVED_FROM           0x00000040      /* File was moved from X */`  
`34 #define IN_MOVED_TO             0x00000080      /* File was moved to Y */`  
`35 #define IN_CREATE               0x00000100      /* Subfile was created */`  
`36 #define IN_DELETE               0x00000200      /* Subfile was deleted */`  
`37 #define IN_DELETE_SELF          0x00000400      /* Self was deleted */`  
`38 #define IN_MOVE_SELF            0x00000800      /* Self was moved */`

Creating traps
--------------

Start editing your incrontab with `incrontab '''-e'''`. Here are some
examples of traps

`# Restart the NTP daemon when its config file changes`  
`/etc/ntp/ntp.conf IN_MODIFY /sbin/service ntpd restart`  
  
`# Run a script with the absolute path to filename as parameter when it changes (and is closed)`  
`/home/nanand/thesis.txt IN_WRITE_CLOSE /home/nanand/bin/log_changes.sh $@/$#`

Here is a full list of wildcards:

`$$ - a dollar sign`  
`$@ - the watched filesystem path `  
`$# - the event-related file name`  
`$% - the event flags (textually)`  
`$& - the event flags (numerically)`

Sources
-------

-   `/usr/share/doc/incron/README`  
    (Ubuntu)
-   `/usr/share/doc/incron-0.5.5/README`  
    (CentOS/RHEL)
