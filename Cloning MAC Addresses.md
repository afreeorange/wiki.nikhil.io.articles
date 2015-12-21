### On RHEL-based systems

If your interface is `eth0`, edit
`/etc/sysconfig/network-scripts/ifcfg-eth0`. If you see a `HWADDR`
param, comment it out and add `MACADDR=xx:xx:xx:xx:xx:xx`. Here's a
sample:

` DEVICE=eth0`  
` BOOTPROTO=dhcp`  
` #HWADDR=00:12:79:A0:59:DE`  
` MACADDR=00:12:79:A0:59:F4`  
` ONBOOT=yes`  
` `  
` # On CentOS 6`  
` BOOTPROTO="dhcp"`

Then restart the network service for this to take effect:

` service network restart`

### On BSD-based systems

If your interface is `igb0`, add a file called `/etc/start_if.igb0` to
specify the cloned MAC:

` ifconfig igb0 ether 00:12:79:45:89:df`

Then reboot your server.



