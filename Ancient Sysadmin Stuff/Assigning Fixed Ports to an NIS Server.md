Configuring Fixed Ports
-----------------------

The motivation is to create better firewalls. You can find out what
ports your server's listening on by issuing `rpcinfo`.

    [admin@server ~]# rpcinfo -p
      program vers proto   port  
       100000    2   tcp    111  portmapper  
       100000    2   udp    111  portmapper  
       100024    1   udp    778  status  
       100024    1   tcp    781  status  
       100004    2   udp   1002  ypserv  
       100004    1   udp   1002  ypserv  
       100004    2   tcp   1005  ypserv  
       100004    1   tcp   1005  ypserv  
       100069    1   udp   1015  fypxfrd  
       100069    1   tcp   1017  fypxfrd  
       100009    1   udp    781  yppasswdd  
       100007    2   udp    958  ypbind  
       100007    1   udp    958  ypbind  
       100007    2   tcp    961  ypbind  
       100007    1   tcp    961  ypbind

You can choose a port that's on that list or come up with your own. I'm
going to use:

* Port **854** for **`ypserv`**
* Port **855** for **`ypxfrd`**

Next, configure `/etc/sysconfig/network` to use these ports. Add this:

    YPSERV_ARGS="-p 854"  
    YPXFRD_ARGS="-p 855"

Restart the `ypserv` and `ypxfrd` services. `rpcinfo` should now show
the fixed ports:

    [admin@server ~]# rpcinfo -p  
      program vers proto   port  
       100000    2   tcp    111  portmapper  
       100000    2   udp    111  portmapper  
       100024    1   udp    778  status  
       100024    1   tcp    781  status  
       100009    1   udp    781  yppasswdd  
       100007    2   udp    960  ypbind  
       100007    1   udp    960  ypbind  
       100007    2   tcp    963  ypbind  
       100007    1   tcp    963  ypbind  
       100004    2   udp    854  ypserv
       100004    1   udp    854  ypserv  
       100004    2   tcp    854  ypserv  
       100004    1   tcp    854  ypserv  
       100069    1   udp    855  fypxfrd
       100069    1   tcp    855  fypxfrd

Sources
-------

* [Securing NIS (RedHat)](http://www.redhat.com/docs/manuals/linux/RHL-9-Manual/security-guide/s1-server-nis.html)
* [Configuring NIS - LinuxHomeNetwork](http://www.linuxhomenetworking.com/wiki/index.php/Quick_HOWTO_:_Ch30_:_Configuring_NIS)
