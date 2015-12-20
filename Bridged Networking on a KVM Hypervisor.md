We will be adding a bridge `vnet0` to interface `eth0`

` [root@otoscope ~]# brctl show`  
` bridge name bridge id         STP enabled interfaces`  
` virbr0      8000.000000000000 yes`

Now add this to `/etc/sysconfig/network-scripts/ifcfg-vnet0`

` DEVICE=vnet0`  
` TYPE=Bridge`  
` BOOTPROTO=dhcp`  
` ONBOOT=yes`

Add `BRIDGE=vnet0` to `/etc/sysconfig/network-scripts/ifcfg-eth0`:

` # Intel Corporation 82579LM Gigabit Network Connection`  
` DEVICE=eth0`  
` BOOTPROTO=dhcp`  
` HWADDR=00:25:90:53:A0:43`  
` ONBOOT=yes`  
` BRIDGE=vnet0`

Restart the network service and add the bridge:

` service network restart`  
` brctl addif vnet0 eth0`

Check:

` [root@otoscope ~]# brctl show`  
` bridge name bridge id          STP enabled interfaces`  
` virbr0      8000.000000000000  yes   `  
` vnet0       8000.00259053a043  no           eth0`

Now add STP with:

`  brctl stp vnet0 on`

Firewall Rules
--------------

Make sure that you've set up the proper forwarding rules with IPTables.
Don't use a general free-for-all like this, though:

` iptables -A FORWARD -m state --state NEW -j ACCEPT`

References
----------

-   [A most **excellent** overview of the `brctl`
    command](http://www.lainoox.com/bridge-brctl-tutorial-linux/).
-   [Here's
    another](http://www.dd-wrt.com/wiki/index.php/Brctl_command).

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
