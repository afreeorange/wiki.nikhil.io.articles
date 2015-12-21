Quick notes
-----------

-   Good idea to send logs to a central, secure log collection server
    -   Better if the server is on another, private network
    -   Second NIC used would be *unnumbered* and in *promiscuous* mode
    -   Port used is **514/UDP**
    -   Frequent target for hackers; PAM, for example, uses a lot
        of syslog.
-   Relay loghosts can also be used. If there are many 'hops', the final
    loghost does not know the source IP
    -   This is fixed by using something like `syslog-ng`

Anatomy of `/etc/syslog.conf`
-----------------------------

Two parts: A *selector* and an *action*.

<table style="width:10%;">
<colgroup>
<col width="0%" />
<col width="3%" />
<col width="3%" />
<col width="3%" />
<col width="0%" />
<col width="0%" />
<col width="0%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"><p><font color="#FF3300">Selector</font></p>
<p>|------------</p></th>
<th align="left"><p>Facility</p></th>
<th align="left"><p>Priority</p></th>
<th align="left"><p><font color="#FF3300">Action</font> |------------</p></th>
<th align="left"><p><code>
* auth&lt;br /&gt;&lt;small&gt;(Security events get logged with this)&lt;/small&gt;
* authpriv&lt;br /&gt;&lt;small&gt;(user access messages use this)&lt;/small&gt;
* cron&lt;br /&gt;&lt;small&gt;(atd and crond daemons)&lt;/small&gt;
* daemon&lt;br /&gt;&lt;small&gt;(other daemon programs without a facility of their own)&lt;/small&gt;
* kern&lt;br /&gt;&lt;small&gt;(kernel messages)&lt;/small&gt;
* lpr&lt;br /&gt;&lt;small&gt;(printing subsystem)&lt;/small&gt;
* mail&lt;br /&gt;&lt;small&gt;(mail system)&lt;/small&gt;
* mark&lt;br /&gt;&lt;small&gt;(used by syslogd to produce timestamps in log files)&lt;/small&gt;
* news&lt;br /&gt;&lt;small&gt;(news system)&lt;/small&gt;
* syslog&lt;br /&gt;&lt;small&gt;(internal syslog messages)&lt;/small&gt;
* user&lt;br /&gt;&lt;small&gt;(for user programs)&lt;/small&gt;
* uucp local0 – local7&lt;br /&gt;&lt;small&gt;(any use; RH uses local7 for boot messages)&lt;/small&gt;
* *&lt;br /&gt;&lt;small&gt;(for all)&lt;/small&gt;
</code></p></th>
<th align="left"><p><code> 
* emerg&lt;br /&gt;&lt;small&gt;(system unavailable)&lt;/small&gt;
* alert&lt;br /&gt;&lt;small&gt;(immediate action required)&lt;/small&gt; 
* crit&lt;br /&gt;&lt;small&gt;(critical condition)&lt;/small&gt;
* err&lt;br /&gt;&lt;small&gt;(error)&lt;/small&gt;
* warning&lt;br /&gt;&lt;small&gt;(what it says)&lt;/small&gt;
* notice&lt;br /&gt;&lt;small&gt;(normal but significant)&lt;/small&gt;
* info&lt;br /&gt;&lt;small&gt;(normal)&lt;/small&gt;
* debug&lt;br /&gt;&lt;small&gt;(debugging info)&lt;/small&gt;
</code><br />
<small>(Importance, descending)</small></p></th>
<th align="left"><p><code>
* /complete/path/of/some/file 
* /dev/console
* -/complete/path/of/some/file&lt;br /&gt;&lt;small&gt;(Don't flush file each time; better performance but risks loss of some log info.)&lt;/small&gt;
* username1[,username2 ...] 
* *&lt;br /&gt;&lt;small&gt;(all logged in users)&lt;/small&gt;
* @remotehost.org
* |/path/to/named/pipe&lt;br /&gt;&lt;small&gt;(To send output to a command you must create a named pipe, say /var/lib/cmd.pipe with the mkfifo command. Then start the command with cmd &lt;/var/lib/cmd.pipe.)&lt;/small&gt;
</code></p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

### Notes & Examples

-   *Cannot* create new facilities. Need to use `local{0,7}`
-   Syslog assumes that the program sending it logs knows how to do so.
-   See [the
    PDF](Media:Logging,_Log_File_Rotation,_and_Syslog_Tutorial.pdf "wikilink")
    for examples of selectors

Logger
------

I use this to quickly test/view a facility or priority

` logger [-p facility.priority] [-t tag] message`

-   The default selector is `user.info`
-   The default tag is `logger`.

Sources
-------

-   [Logging, Log File Rotation, and Syslog
    Tutorial](Media:Logging,_Log_File_Rotation,_and_Syslog_Tutorial.pdf "wikilink")
-   [System Logging Explained in
    Linux](http://linuxhelp.blogspot.com/2005/09/system-logging-explained-in-linux.html)
-   [Unix/Linux System Administration - Syslog
    Module](http://fog.ccsf.cc.ca.us/~gboyd/cs260a/online/syslog/introduction.html)
