Installing Samba
----------------

On CentOS, issue the following:

` yum install samba samba-client`

You can see all the files installed using `rpm -qd`. The chief one is
`/etc/samba/smb.conf`

Creating a Samba users
----------------------

Much like adding users to NIS, you need to add a user the Linux way and
tell samba about this user.

` useradd -g sambausersgroup sambauser`  
` passwd sambauser`

Then tell Samba about this user:

`smbpasswd -a sambauser`

The two passwords can (obviously) be different. You may get an error
along the lines of:

` account_policy_get: tdb_fetch_uint32 failed for field `<integer>

From what I could learn, this is 'normal' for the first time `smbpasswd`
is run and should not appear the next time.

Setting up `smb.conf`
---------------------

Make sure you have at least these lines under `[Global]` (adapt to your
specific case):

` workgroup = HOME`  
` server string = IT Support - Samba Version %v`  
` netbios name = HOME Support System`  
` `  
` security = user`  
` passdb backend = tdbsam`  
` encrypt passwords = yes`  
` load printers = no`

At the least, make sure that the `encrypt passwords` option is set to
`yes`. Although I will restrict access with IPTables, `smb.conf` itself
allows you to restrict access to resources on a share-by-share basis.

### Adding a share

In this example, I will create two shares: one read-only and another
read/write for `sambauser` created before.

` [The Read-Only Share]`  
`       comment = This is a test read-only share`  
`       path = /home/sambauser/testreadshare`  
`       browseable = yes `  
`       guest ok = no`  
`       writable = no`  
`       valid users = sambauser`

` [The Read/Write Share]`  
`       comment = sambauser can read and write to this share `  
`       path = /media/uploads`  
`       browseable = Yes`  
`       guest ok = No`  
`       writeable = Yes`  
`       write list = sambauser`  
`       valid users = sambauser`

**This is important**: *Linux system permissions take precedence over
Samba permissions*. For example if a directory does not have Linux write
permission, setting samba `writeable = Yes` will not allow to write to
shared directory / share.

### Verify the correctness of `smb.conf`

Issue `testparm` and you should see something like:

` [root@localhost ~]# testparm`  
` Load smb config files from /etc/samba/smb.conf`  
` Processing section "[IPC$]"`  
` WARNING: No path in service IPC$ - making it unavailable!`  
` NOTE: Service IPC$ is flagged unavailable.`  
` Processing section "[Tech Shed]"`  
` Loaded services file OK.`  
` Server role: ROLE_STANDALONE`  
` Press enter to see a dump of your service definitions`

Hitting enter will dump your share definitions.

### Configure IPTables

The relevant ports are [**UDP (137, 138)** and **TCP (139, 445)** (click
for further elaboration)](NetBIOS_and_Samba_Ports "wikilink"). Here's a
sample

`iptables -A INPUT -p udp -m multiport --dport 137,138 -m state --state NEW -j ACCEPT`  
`iptables -A INPUT -p tcp -m multiport --dport 139,445 -m state --state NEW -j ACCEPT`  
`iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`

Starting and monitoring Samba
-----------------------------

` service smb start`

If you've changed your `smb.conf` file and only want to reload
new/modified shares:

` service smb reload`

This starts `smbd` (the Samba daemon) and `nmbd` (the NetBIOS
nameserver). You can check if Samba is listening to the correct ports by
issuing `netstat -tulpn`.

### Log files

Default log files are `/var/log/samba/{smbd.log, nmbd.log}`. These may
help troubleshoot any issues with startup or share connectivity.

### Connecting to a share

Get a listing of the shares on a host using `smbclient` (installed as
the package `samba-client`)

` smbclient -L "//hostname.example.com" -U sambauser`

Mount a share using `mount` as follows:

`mount -t cifs -o user=sambauser "//hostname.example.com/My Uploads" /mnt/uploads`

You can add this to your `/etc/fstab` too!

### Listing open files

Simply issue `smbstatus` to see all mounted shares. Here's some sample
output:

` Samba version 3.0.33-3.15.el5_4.1`  
` PID     Username      Group         Machine                        `  
` -------------------------------------------------------------------`  
` 26197   support       support       dhcpw80ff9676 (19.67.90.10)`  
` `  
` Service      pid     machine       Connected at`  
` -------------------------------------------------------`  
` Tech Shed    26197   dhcpw80ff9676  Mon Apr 26 08:38:59 2010`  
` `  
` Locked files:`  
` Pid          Uid        DenyMode   Access      R/W        Oplock           SharePath   Name   Time`  
` --------------------------------------------------------------------------------------------------`  
` 26197        501        DENY_NONE  0x100001    RDONLY     NONE             /media/techshed   .   Mon Apr 26 08:39:07 2010`

Common Issues
-------------

From what I've experienced and read, adding your hostname to
`/etc/hosts` either speeds up or solves many issues with `smbd`.

A more egregious problem was with my IPTables config. Samba tries to
access port 631 even though I set `load printers` to **no** in my config
above. Since my IPTables blocking any unnecessary outputs as well, this
led to a point where issuing `service smb restart` would cause the
system to stall at:

` Starting SMB Services`

I could start `nmbd` manually by issuing `nmbd -D`. When I opened up
another terminal and checked the status of the SMB daemon, it was listed
as running *even though it was stalled at "Starting..." in the previous
terminal*!

I solved this after much psychological torture by adding these lines to
`smb.conf`:

` show add printer wizard = no`  
` printing = none`  
` printcap name = /dev/null`  
` disable spoolss = yes`

### Issues with PoPToP

The problem with trying to access an SMB share through a PPTP tunnel [is
elaborated here](http://poptop.sourceforge.net/dox/howto1.html#6.6).
This is what I added to make things work (to `smb.conf`)

` domain master = yes`  
` domain logons = yes`

Integrating OpenDirectory with Samba
------------------------------------

### On OpenDirectory Master (10.4 Server)

-   Enable, start the Windows service
-   Make it a PDC (Primary Domain Controller) instead of Standalone
    Server
-   Then entered this in Settings &gt; General

` Description   : OpenDirectory Master`  
` Computer Name : directory`  
` Domain        : HOME`

-   Under Settings &gt; Access, uncheck LAN Manager (used for W95
    support, insecure)
-   In firewall, enable ports 137, 138, 139 and 445

### On Linux Box

Use `authconfig-tui` to configure with LDAP. Make sure you check the
"Use LDAP" and "Use LDAP Authentication" boxes. Type in the name of the
server and the search base. This effectively makes changes to
`/etc/nsswitch.conf` and `/etc/pam.d/system-auth`. You know when the
LDAP lookups are working when your server can pull directory information
like this:

` [root@tiner etc]# id nanand`  
` uid=40010(nanand) gid=20(games) groups=20(games),80(admin),1026(sheffield)`

I then added this to `smb.conf` ("`testuser`" is the group I'd like to
restrict access to.)

` workgroup = HOME`  
` security = domain`  
` encrypt passwords = yes`  
` password server = directory`  
` `  
` [public]`  
` comment = Test Share`  
` path = /home/support`  
` public = yes`  
` writable = yes`  
` valid users = @testuser`  
` force group = @HOME\testuser`

Then set valid permissions on the folder you've shared

` chown -R nobody:testuser /home/support`

Then join the Samba box to the domain.

` [root@tiner home]# net join -S directory -U diradmin`  
` Password:`  
` Joined domain HOME.`  
` [root@tiner home]# service smb restart`

Test and enjoy.
