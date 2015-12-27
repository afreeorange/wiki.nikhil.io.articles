The following are my notes of installing PoPToP on **example.com** (RHEL5, i386)

Pre-Flight Check
----------------

Check if your kernel supports MPPE (Microsoft Point to Point
Encryption).

    [root@example ~]# modprobe ppp-compress-18 && echo OK  
    OK

Usually, kernels 2.6.15 and above support MPPE

Install the necessary packages
------------------------------

First install PPP:

    yum install ppp

Next install PoPToP:

    rpm -ivH http://poptop.sourceforge.net/yum/stable/rhel5/i386/pptpd-1.3.4-1.rhel5.1.i386.rpm

Check if everything went well with

    rpm -qa pptpd -d

Configure PPTPD
---------------

You'll configure the behaviour of your PPTP server in two locations:

*   All passwords and connection params will be in `/etc/ppp`.
*   The behaviour of the server itself will be controlled with
    `/etc/pptpd.conf`

### Edit /etc/ppp/options.pptpd to use DNS and require encryption

Configuring `/etc/ppp/options.pptpd` is rather easy. I only checked the
file to make sure that MPPE was required for the connection, that the
old-style (and insecure) PAP and CHAP was not enabled, and added the
following lines:

    ms-dns 19.27.17.20  
    ms-dns 19.27.12.21

### Edit /etc/ppp/chap-secrets to manage users

This is very simple too. To add a new user, add a line that looks like:

    # Secrets for authentication using CHAP  
    # client  server  secret  IP addresses  
    tomc * C@saVa\t *

In this case, the user `tomc` would have the password `C@saVa\t`.

### Edit /etc/pptpd.conf to disburse IP addresses

I only edited/added the following lines. They tell the server to sustain
a maximum of 30 connections and use the remote (client) IP range
`10.9.0.2` to `10.9.0.31`.

    connections 30  
      
    localip 19.27.18.14  
    remoteip 10.9.0.2-31

Configure logging
-----------------

Before starting the PPTPD daemon (how's that for a tautology), it would
be nice to log things to a file to keep track of users and diagnose
connection problems. For this, we can use `syslogd`. Add this to
**daemon** facility in `/etc/syslogd.conf`:

    daemon.*  /var/log/pptpd.log  

Then restart the syslog daemon:

    killall syslogd  
    /sbin/syslogd

Configure the local system to allow connectiong and IP forwarding
-----------------------------------------------------------------

### Configure iptables rules

I have a very restrictive set of iptables rules and needed all of these.
Your mileage may vary. PPTPD uses port 1723 and *protocol* 47. In this
snippet, my `$EXTERNAL_INTERFACE` variable is set to `eth1`.

    # Allow PPTP. Note: it's not _port_ 47 but _protocol_ 47 ("GRE", by Cisco)  
    iptables -A INPUT -i ppp+ -j ACCEPT  
    iptables -A OUTPUT -o $EXTERNAL_INTERFACE -j ACCEPT  
      
    iptables -A INPUT  -p tcp --dport 1723 -j ACCEPT  
    iptables -A OUTPUT -p tcp --sport 1723 -j ACCEPT  
    iptables -A INPUT  -p 47 -j ACCEPT  
    iptables -A OUTPUT -p 47 -j ACCEPT  
      
    iptables -A FORWARD -i ppp+ -o $EXTERNAL_INTERFACE -m state --state NEW -j ACCEPT  
    iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT  
    iptables -A POSTROUTING -t nat -j MASQUERADE

### IP forwarding

You might also have to enable IP forwarding. Check if this is already
enabled by issuing:

    [root@example ~]# sysctl net.ipv4.ip_forward  
    net.ipv4.ip_forward = 0  
      
    or  
      
    [root@example ~]# cat /proc/sys/net/ipv4/ip_forward  
    0  

If you see a **1**, you're good. If not, you can enable it on the fly
like so:

    [root@example ~]# sysctl -w net.ipv4.ip_forward=1  
      
    or  
      
    [root@example ~]# echo 1 > /proc/sys/net/ipv4/ip_forward  
    

If you do this, make sure that `/etc/sysctl.conf` has
`net.ipv4.ip_forward` set to **1**. Then restart the network service:

    [root@example ~]# sysctl -p /etc/sysctl.conf  
      
    or  
      
    [root@example ~]# service network restart

Start the PPTPD service (finally!)
----------------------------------

    service pptpd start

And you should be good to go. Diagnose network problems with nmap,
traceroute, etc. You may also want to start it at reboot

    chkconfig --level 345 pptpd on

Resources
---------

[Poptop Installation guide](http://poptop.sourceforge.net/dox/howto1.html)

