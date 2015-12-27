Quick notes
-----------

*   Good idea to send logs to a central, secure log collection server
    *   Better if the server is on another, private network
    *   Second NIC used would be *unnumbered* and in *promiscuous* mode
    *   Port used is **514/UDP**
    *   Frequent target for hackers; PAM, for example, uses a lot
        of syslog.
*   Relay loghosts can also be used. If there are many 'hops', the final
    loghost does not know the source IP
    *   This is fixed by using something like `syslog-ng`

Anatomy of `/etc/syslog.conf`
-----------------------------

Two parts: A *selector* and an *action*. TODO: Convert to HTML...

    {|class="wikitable" width="100%"
    !colspan="2"|<font color="#FF3300">Selector</font>
    |------------
    !width="33%"|Facility
    !width="33%"|Priority
    !width="33%"|<font color="#FF3300">Action</font>
    |------------
    |
    <code>
    * auth<br /><small>(Security events get logged with this)</small>
    * authpriv<br /><small>(user access messages use this)</small>
    * cron<br /><small>(atd and crond daemons)</small>
    * daemon<br /><small>(other daemon programs without a facility of their own)</small>
    * kern<br /><small>(kernel messages)</small>
    * lpr<br /><small>(printing subsystem)</small>
    * mail<br /><small>(mail system)</small>
    * mark<br /><small>(used by syslogd to produce timestamps in log files)</small>
    * news<br /><small>(news system)</small>
    * syslog<br /><small>(internal syslog messages)</small>
    * user<br /><small>(for user programs)</small>
    * uucp local0 – local7<br /><small>(any use; RH uses local7 for boot messages)</small>
    * *<br /><small>(for all)</small>
    </code>
    |valign="top"|
    <code> 
    * emerg<br /><small>(system unavailable)</small>
    * alert<br /><small>(immediate action required)</small> 
    * crit<br /><small>(critical condition)</small>
    * err<br /><small>(error)</small>
    * warning<br /><small>(what it says)</small>
    * notice<br /><small>(normal but significant)</small>
    * info<br /><small>(normal)</small>
    * debug<br /><small>(debugging info)</small>
    </code><br /><small>(Importance, descending)</small>
    |valign="top"|
    <code>
    * /complete/path/of/some/file 
    * /dev/console
    * -/complete/path/of/some/file<br /><small>(Don't flush file each time; better performance but risks loss of some log info.)</small>
    * username1[,username2 ...] 
    * *<br /><small>(all logged in users)</small>
    * @remotehost.org
    * |/path/to/named/pipe<br /><small>(To send output to a command you must create a named pipe, say /var/lib/cmd.pipe with the mkfifo command. Then start the command with cmd </var/lib/cmd.pipe.)</small>
    </code>
    |}


### Notes & Examples

*   *Cannot* create new facilities. Need to use `local{0,7}`
*   Syslog assumes that the program sending it logs knows how to do so.
*   See [the
    PDF](Media:Logging,_Log_File_Rotation,_and_Syslog_Tutorial.pdf "wikilink")
    for examples of selectors

Logger
------

I use this to quickly test/view a facility or priority

    logger [-p facility.priority] [-t tag] message

*   The default selector is `user.info`
*   The default tag is `logger`.

Sources
-------

*   [Logging, Log File Rotation, and Syslog Tutorial](Media:Logging,_Log_File_Rotation,_and_Syslog_Tutorial.pdf "wikilink")
*   [System Logging Explained in Linux](http://linuxhelp.blogspot.com/2005/09/system-logging-explained-in-linux.html)
*   [Unix/Linux System Administration - Syslog Module](http://fog.ccsf.cc.ca.us/~gboyd/cs260a/online/syslog/introduction.html)
