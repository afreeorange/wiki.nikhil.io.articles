Pre-Flight
----------

*   SNMP is **UDP** on port **161**
*   Zenoss provides [a lot of useful
    documentation](http://community.zenoss.org/community/documentation/official_documentation?view=overview).
*   The following ports need to be allowed *to* the **server**:

| Port | Protocol |     Description      |
|------|----------|----------------------|
| 8080 | HTTP     | Zenoss Web Interface |
|  514 | UDP      | Inbound syslog       |
|  162 | UDP      | Inbound SNMP Traps   |

Installation
------------

### Zenoss Server

**Note**: Zenoss does *not* like special characters in the MySQL root
password.

```bash
# Install necessary packages; MySQL and net-snmp are required  
yum -y install mysql-server net-snmp net-snmp-utils gmp libgomp libgcj liberation-fonts  
  
# Configure MySQL  
  
# Install Zenoss  
mkdir zenoss.temp; cd zenoss.temp  
wget http://downloads.sourceforge.net/zenoss/zenoss-3.2.0.el5.i386.rpm
rpm -ivh zenoss-3.2.0.el5.i386.rpm  
service zenoss start
```

This sets up *everything*. Go get some tea; this shall take a
while. Then navigate to `http://<FQDN>:8080` to finish setup. [This
book](http://www.amazon.com/Zenoss-Core-Network-System-Monitoring/dp/1849511586)
is your friend, friend.

Uninstallation
--------------

Navigate to the directory where Zenoss was installed.

    [root@zenoss zenoss]# ./uninstall

Miscellaneous
-------------

*   Edit `/opt/zenoss/etc/zope.conf` and edit the `address` under `<http-server>`.
*   Don't change the default directory (`/usr/local/zenoss`)
*   Changed zSnmpVer to v2c under "Configuration Properties" for Device Class
*   Reinstalled Zenoss for HDD monitoring
*   [A most excellent overview of SNMP](http://www.linuxhomenetworking.com/wiki/index.php/Quick_HOWTO_:_Ch22_:_Monitoring_Server_Performance#SNMP_Version_3).
    This is probably all you need to get started.
*   Change admin password like so:

        su - zenoss  
        /usr/local/zenoss/zenoss/bin/zenpass

*   If you have a non-default port, your email alerts will still use the
    default (8080). To change this, edit `$ZENHOME/etc/zenactions.conf`,
    add a line like so, and restart Zenoss

        zopeurl      http://servername.tld:9090
