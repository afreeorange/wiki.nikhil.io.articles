Downloaded the [original firmware just in
case](http://www.asus.com/us/Networking/RTN66U/HelpDesk_Download/). Then
logged into router and backed up config in "Administration" →
"Restore/Save/Upload Setting".

Got the enhanced firmware via [this
page](http://www.lostrealm.ca/tower/node/79) (scroll to the bottom).
Verified MD5.

Updated with new firmware. Rebooted. All was well.

Enabled manual assignment on "LAN" → "DHCP Server" and added devices and
names. This wasn't working before. Yay.

### [DNSMasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)

Edit `/etc/hosts.dnsmasq` to add entries. Found
[this](http://www.dd-wrt.com/wiki/index.php/DNSMasq_-_DNS_for_your_local_network_-_HOWTO)
to be a nice, quick guide.

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
