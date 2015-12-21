Installation
------------

1.  Downloaded the [original firmware just in
    case](http://www.asus.com/us/Networking/RTN66U/HelpDesk_Download/).
    Then logged into router and backed up config in "Administration" →
    "Restore/Save/Upload Setting".
2.  Got the enhanced firmware via [this
    page](http://www.lostrealm.ca/tower/node/79) (scroll to the bottom).
    Verified MD5.
3.  Updated with new firmware. Rebooted via GUI. All was well.

Enabled manual assignment on "LAN" → "DHCP Server" and added devices and
names. This wasn't working before. Yay.

[Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)
--------------------------------------------------------

SSHed into router, created these files:

` /jffs/configs/dnsmasq.conf.add`  
` /jffs/configs/hosts.dnsmasq`

Added this to `dnsmasq.conf.add`:

` addn-hosts=/jffs/configs/hosts.dnsmasq`

And whatever else to the hosts file. Restarted the service:

` ps | grep dnsmasq`  
` kill -9 `<pid>  
` dnsmasq --log-async`

Resources
---------

-   [Quick guide to
    Dnsmasq](http://www.dd-wrt.com/wiki/index.php/DNSMasq_-_DNS_for_your_local_network_-_HOWTO)
-   [Custom Config and Postconf files in
    ASUSWRT](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files)
