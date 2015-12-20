This is a quick overview of `logrotate` which... umm... rotates log
files. This is specific to CentOS/RHEL and was written for release 5.5.

Pertinent Files & Directories
-----------------------------

<table style="width:10%;">
<colgroup>
<col width="4%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"><p>File/Directory</p></th>
<th align="left"><p>Purpose |-------</p></th>
<th align="left"><p><code>/usr/sbin/logrotate</code></p></th>
<th align="left"><p>The command itself. |-------</p></th>
<th align="left"><p><code>/etc/cron.daily/logrotate</code></p></th>
<th align="left"><p>Bash script that executes the <code>logrotate</code> command every day. |-------</p></th>
<th align="left"><p><code>/etc/logrotate.conf</code> and <code>/etc/logrotate.d/</code></p></th>
<th align="left"><p>Rotation defaults if they are not defined for specific daemons. You can add rotate configs to this file, or put them in <code>/etc/logrotate.d</code> |-------</p></th>
<th align="left"><p><code>/var/lib/logrotate.status</code></p></th>
<th align="left"><p>Show when the specified log files were last rotated. |-------</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Logrotate Options
-----------------

Quite simple, really. The `man` page elucidates all available options.
Here's a quick table of what the most commonly used ones do.

<table style="width:10%;">
<colgroup>
<col width="3%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"><p>Option</p></th>
<th align="left"><p>Purpose |-------</p></th>
<th align="left"><p><code>rotate &lt;number&gt;</code></p></th>
<th align="left"><p>Keep specified number of logfiles before they are deleted and/or emailed to admin |-------</p></th>
<th align="left"><p><code>size=&lt;number&gt;</code></p></th>
<th align="left"><p>Rotate logs when they reach this size.</p>
<ul>
<li>This is done <em>without regard</em> for the last rotated time.</li>
<li>Use <code>minsize &lt;number&gt;</code> if you'd like to balance both time and size.</li>
<li>Bytes are used without a specifier. 100k, 100M are also fine.</li>
</ul>
<p>|-------</p></th>
<th align="left"><p><code>daily</code></p></th>
<th align="left"><p>Rotate daily. This is the default minimum granularity, unless you move <code>/etc/'''cron.daily'''/logrotate</code> to <code>/etc/'''cron.hourly'''/logrotate</code> |------</p></th>
<th align="left"><p><code>weekly</code></p></th>
<th align="left"><p>Rotate weekly |------</p></th>
<th align="left"><p><code>monthly</code></p></th>
<th align="left"><p>Rotate monthly |------</p></th>
<th align="left"><p><code>notifempty</code></p></th>
<th align="left"><p>Don't rotate if empty (the reverse, <code>ifempty</code>, is default) |------</p></th>
<th align="left"><p><code>missingok</code></p></th>
<th align="left"><p>If log file is missing for some reason, move on without error |------</p></th>
<th align="left"><p><code>compress</code></p></th>
<th align="left"><p>Compress logfile after rotation.</p>
<ul>
<li>Default is <code>gzip</code> with <code>-9</code> (maximum) compression.</li>
<li>The choice of compression program can be changed with <code>compresscmd</code> (e.g. <code>compresscmd bzip2</code>).</li>
<li>To pass options (like compression level), use <code>compressoptions</code>.</li>
<li>To change extension of compressed file, use <code>compressext</code></li>
</ul>
<p>|------</p></th>
<th align="left"><p><code>dateext</code></p></th>
<th align="left"><p>Use YYYYMMDD format instead of just tacking on numbers (like <code>.0</code>, <code>.1</code> and so on) to the rotated files. |------</p></th>
<th align="left"><p><code>mail &lt;recipient&gt;</code></p></th>
<th align="left"><p>Email recipient the file that will be deleted after a rotation cycle |------</p></th>
<th align="left"><p><code>create &lt;mode&gt; &lt;owner&gt; &lt;group&gt;</code></p></th>
<th align="left"><p>Create logfiles with the specified permissions and owner:group attributes. |------</p></th>
<th align="left"><p><code>olddir &lt;directory&gt;</code></p></th>
<th align="left"><p>Move all but the newest log file to this directory (nice to keep things organized)</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

### Other (kinda important) options

You'll see these used with Apache, for instance.

<table style="width:10%;">
<colgroup>
<col width="2%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"><p>Option</p></th>
<th align="left"><p>Purpose |-------</p></th>
<th align="left"><p><code>prerotate</code></p></th>
<th align="left"><p>Execute the a script <em>before</em> rotating a log. Should end this with <code>endscript</code>. |-------</p></th>
<th align="left"><p><code>postrotate</code></p></th>
<th align="left"><p>Execute the a script <em>after</em> rotating a log. Should end this with <code>endscript</code>. |-------</p></th>
<th align="left"><p><code>sharedscripts</code></p></th>
<th align="left"><p>Make sure that the script(s) specified in <code>prerotate</code> and/or <code>postrotate</code> run just <em>once</em> (i.e. not for <em>every</em> logfile) |------</p></th>
<th align="left"><p><code>delaycompress</code></p></th>
<th align="left"><p>Don't compress yesterday's logfile (if daily... you get the picture)</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Example
-------

Here's a real-world application of the above. I want to rotate an
already huge log of OpenVPN connections (\~32M in size) like so:

-   I want to maintain **60 days** worth of logs
-   Logfile **size doesn't matter** to me
-   The files should be rotated in **YYYYMMDD** format
-   They should be **compressed** (the default `gzip -9` is fine)
-   They should be **organized**; older logfiles should be **in a
    separate directory**

The logfile is at `/var/log/openvpn/connections.log`. Here's the
configuration file I created for the above. It's
`/etc/logrotate.d/openvpn`:

` /var/log/openvpn/connections.log {`  
`     daily`  
`     rotate 60`  
`     dateext`  
`     compress`  
`     olddir /var/log/openvpn/old`  
` `  
`     nomail`  
`     missingok`  
`     notifempty`  
`     delaycompress`  
`     create 640 root root`  
` }`

To test this, I run:

` logrotate -d /etc/logrotate.d/openvpn`

The `-d` switch runs it in verbose, debug, dry-run mode (i.e. nothing
actually happens.)

Other Stuff
-----------

### Leopard and Snow Leopard

These use a new utility called `newsyslog`. It is invoked every minute
by:

` /System/Library/LaunchDaemons/com.apple.newsyslog.plist`

The config file for this is `/etc/newsyslog.conf` and is kinda nicer
than `logrotate`'s config :)

Sources
-------

-   [Logrotate
    Examples](http://www.thegeekstuff.com/2010/07/logrotate-examples/)
-   [Using logrotate to manage log
    files](http://linuxers.org/howto/howto-use-logrotate-manage-log-files)
-   [Rotating Linux log
    files](http://www.ducea.com/2006/06/06/rotating-linux-log-files-part-2-logrotate/)

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
