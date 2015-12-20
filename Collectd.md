Amazing little thing that collects data on interfaces, processes,
daemons and tons of other stuff on your system. You can even centralize
log collection! Some details on this install log:

-   RHEL/CentOS 5.5 64-bit system
-   `collectd` v4.10 from EPEL
-   Configured to collect data locally

Installation
------------

` yum install collectd collectd-apache collectd-rrdtool`

Problem:

` collectd-4.10.0-4.el5.i386 from epel has depsolving problems`  
`   --> Missing Dependency: libpython2.4.so.1.0 is needed by package collectd-4.10.0-4.el5.i386 (epel)`  
` Error: Missing Dependency: libpython2.4.so.1.0 is needed by package collectd-4.10.0-4.el5.i386 (epel)`

This file, however, exists:

` [root@support other_scripts]# `**`locate` `libpython2.4.so.1.0`**  
` /usr/lib64/libpython2.4.so.1.0`

The issue is that the 64-bit repo has *both* 32-bit and 64-bit versions.
So you need to be a little more explicit:

`   yum install collectd`*`.x86_64`*` collectd-apache collectd-rrdtool`

Done!

Configuration
-------------

Make a copy and edit the config file `/etc/collectd.conf`:

` cp /etc/collectd.conf /etc/collectd.conf.original`

Add the following for base config that will monitor CPU, Interface, Load
and Memory:

` Hostname    "support.eng.uiowa.edu"`  
` FQDNLookup   true`  
` BaseDir     "/var/lib/collectd"`  
` PIDFile     "/var/run/collectd.pid"`  
` PluginDir   "/usr/lib64/collectd"`  
` TypesDB     "/usr/share/collectd/types.db"`  
` Interval     10  `  
` ReadThreads  5`

**This is bloody important**: Observe the `lib64` in the `PluginDir`
above for 64-bit installs. I wasted 2 hours of my life wondering why the
damn thing didn't work and started blaming the packagers.

At the bottom of the file, add this:

` include "/etc/collectd.d/*.conf"`

This basically keeps things organized and modular w.r.t. new plugins
(all of which can be defined in `collectd.conf` itself).

Now start the service and watch as
`/var/lib/collectd/support.eng.uiowa.edu` gets populated.

Plugin Management
-----------------

### Overview

Plugins are basically `.so` files (or executable code) which are used by
the `collectd` daemon if the are enabled. They are configured
individually via XML-style directives in either `collectd.conf` or in
`/etc/collectd.d` (see below).

### Using existing plugins

Just uncomment the corresponding `LoadPlugin` in `/etc/collectd.conf`
and restart the service!

### Adding new plugins

Like the config file mentions, there are a few plugins which you will
have to install manually. For example, I'm interested in the Apache and
RRDtool plugins. These are not bundled with `collectd` by default, but
are available via `yum`:

` yum install collectd-apache collectd-rrdtool`

They install shared object files in `/usr/lib64/collectd` (on a 64-bit
system) and a config file in `/etc/collectd.d`.

The upshot is that they include `LoadPlugin` directives, so don't touch
the `collectd.conf` file to enable them. Remember the `include`
directive we added above.

### Configuring Plugins

Read the `man` page for `collectd.conf` or the documentation on the
plugin's website.

Graphing
--------

You would use `rrdtool` for this. It deserves its own Wiki article(s).
But for instant gratification, install the `collectd-web` package:

` yum -y install collectd-web`

This installs a bunch of CGI scripts in
`/usr/share/collectd/collection3`. You can now move them to your Apache
cgi-bin to run them

` cp -rv /usr/share/collectd/collection3 /var/www/cgi-bin/`  
` chmod -R 755 /var/www/cgi-bin/collection3/bin`

If you've configured Apache properly, you can now navigate to
<http://hostname/cgi-bin/collection3/bin/index.cgi> and enjoy your
graphs! Remember that you can still do a ton of amazing graph-related
things with `rrdtool`

Sources
-------

-   [Graphing collectd statistics with
    Visage](http://holmwood.id.au/~lindsay/2009/09/08/graphing-collectd-statistics-in-the-browser-with-visage/)
    (Ruby, JSON)
-   [RRD
    Tutorial](http://oss.oetiker.ch/rrdtool/tut/rrdtutorial.en.html)

[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
