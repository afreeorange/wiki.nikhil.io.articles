Background
----------

*   Vanilla installation of Xen v3.0.3 on `hypervisor.example.com`.
    All defaults.
*   Platform is CentOS 5. Paravirtualization [is not supported on CentOS
    6](https://www.centos.org/modules/newbb/viewtopic.php?topic_id=37151).
    It's possible to [make it work](http://www.howtoforge.com/virtualization-with-xen-on-centos-6.2-x86_64-paravirtualization-and-hardware-virtualization),
    but I think you should get a newer processor and run KVM if using
    CentOS 6 to save yourself the trouble.

Glossary
--------

Not meant to be complete.


|                                  Term                                  |                                                                              Explanation                                                                              |
|------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Type I Hypervisor                                                      | Runs directly on hardware. Virtual machines don't know they're virtualized.                                                                                           |
| Type II Hypervisor                                                     | Hypervisor (Xen) runs in OS (RHEL/CentOS). The virtual machines ''know'' they're being run in a virtual environment                                                   |
| HVM ("Hardware Virtual Machine" or "Hardware-assisted Virtualization") | Not entirely sure about this. Certain processor technology (e.g. Intel VT-x) allows "complete simulation of underlying hardware." VMs don't know they're virtualized. |
| `dom0`                                                                 | The hypervisor itself                                                                                                                                                 |
| `domU`                                                                 | A single virtual instance                                                                                                                                             |
| `xm`                                                                   | Xen-provided tool to manage domU's                                                                                                                                    |
| `virsh`                                                                | A Red Hat-designed shell to manage VM's. Differs from `xm` in that it can manage QEMU and HVM-based domU's as well since it's based on the `libvirt` API.             |
| `virt-install` and `virt-manager`                                      | Management and provisioning tools based on `libvirt`/                                                                                                                 |

Installation
------------

    yum groupinstall Xen  
    yum install python-virtinst qemu*

The first installs the Xen-enabled kernel, Xen daemon, virtualization
libraries, etc. Make sure that (a) SELinux is disabled, and (b) that you
reboot into the Xen kernel before doing anything else.

The First VM
------------

### Preparing the `dom0`

*   My VMs will be running CentOS 6. So I
    [downloaded](http://mirror.anl.gov/pub/centos/6/isos/) and
    loop-mounted the latest CentOS 6 ISO. I then offered the mount via
    HTTP for VM installation.
*   I then created logical volumes for use as storage by the VMs. You
    can [also format and use disk
    images](http://www.chrisabernethy.com/how-to-resize-a-xen-virtual-disk/).

### Creating the VM

`virt-manager` is the easiest way to do things. You can do a
command-line install via `virt-install`. Here's a sample command that
creates a 64-bit VM called "devel1" running CentOS 6 with two virtual
CPUs and 1.2GB of RAM. Observe that I explicitly specify the MAC
address.

    virt-install \  
    --name=devel1 \  
    --arch=x86_64 \  
    --vcpus=2 --check-cpu \  
    --ram=1200 \  
    --disk path=/dev/xenspace/devel1 \  
    --mac=00:0C:29:1A:98:D5 \  
    --os-type=linux \  
    --os-variant=rhel6 \  
    --location=http://hypervisor.example.com/install/6/x86_64/ \  
    --debug \  
    --nographics

Once the VM is installed, it's a good idea to save the kickstart files.
Here's a sample:

    # Modified by Nikhil Anand 
    install
    url --url http://hypervisor.example.com/install/6/x86_64/
    lang en_US.UTF-8
    keyboard us
    network --device eth0 --bootproto dhcp
    rootpw --iscrypted $1$9P2b0WZe$CSd.fBGCVjjUfzlZ6m5Rk1
    firewall --enabled --port=22:tcp
    authconfig --enableshadow --enablemd5
    selinux --enforcing
    timezone --utc America/Chicago
    bootloader --location=mbr --driveorder=xvda
    # The following is the partition information you requested
    # Note that any partitions you deleted are not expressed
    # here so unless you clear all partitions first, this is
    # not guaranteed to work
    clearpart --linux --drives=xvda
    part /boot --fstype ext3 --size=100 --ondisk=xvda
    part pv.6 --size=0 --grow --ondisk=xvda
    volgroup VolGroup00 --pesize=32768 pv.6
    logvol / --fstype ext3 --name=LogVol00 --vgname=VolGroup00 --size=1024 --grow
    logvol swap --fstype swap --name=LogVol01 --vgname=VolGroup00 --size=528 --grow --maxsize=1056

    %packages
    @base
    @core
    keyutils
    iscsi-initiator-utils
    trousers
    fipscheck
    device-mapper-multipath

If you ever wanted to reinstall the VM, you can now append a flag with
the (HTTP downloadable) path to the kickstart file:

    -x "ks=http://hypervisor.example.com/kickstarts/centos-6.ks"

HVM Support
-----------

You can find if your processor supports HVM by issuing

    egrep '^flags.*(vmx|svm)' /proc/cpuinfo

Network Topologies
------------------

Xen offers the following:

*   Bridged
*   NAT-ted
*   Routed

It's unusual (and crazy) to use all three on a given dom0 instance. The
default is bridged networking. The `brctl` command is used to manage
network bridges.

In our case, the router hands out DHCP leases depending on MAC
addresses. This is why I didn't have to do anything other than specify
the MAC address in a domU's config:

    vif = [ "mac=00:50:56:78:0a:1b,bridge=xenbr0,script=vif-bridge" ]

More exotic configurations are possible. You can, for example, specify
two virtual interfaces (`vif`'s), with public and private IPs. In this
case, the `route` and `iptables` commands become important, since you'll
have to set up routes and masquerading.

Edit `/etc/xen/xend-config.sxp` to set up these configs. For instance,
if you only had a routed config, you'd comment out every other
`network-script` and `vif-script` other than these:

    #(network-script network-route)  
    #(vif-script     vif-route)

PyGRUB
------

`virt-install` removes the `kernel` and `ramdisk` lines from a domU's
config file and adds this instead:

    bootloader = "/usr/bin/pygrub"

PyGRUB itself will look for the [*first partition or LVM container* that
contain the kernel and init image](http://wiki.xen.org/xenwiki/PyGrub).

I made an error of using the [CentOS project-supplied kernel and
ramdisk](http://mirror.centos.org/centos/5/os/x86_64/images/xen/), which
were good for an install, but useless when the domU was rebooted.
They're built specifically for installation :)

"Could not connect to localhost:8000"
-------------------------------------

You may see this when using `virt-install` or `virt-manager`. Edit
`/etc/xen/xend-config.sxp` and make sure these lines are uncommented:

    (xend-http-server yes)  
    (xend-port 8000)  
    (xend-address localhost)

And restart the Xen daemon.

Logging
-------

You're supposed to be able to edit `/etc/sysconfig/xend`, uncomment this
line and see logs in `/var/log/xen/console`

    XENCONSOLED_LOG_DIR=/var/log/xen/console

Didn't work for me.

Miscellaneous
-------------

### "Guest name already in use"

    virsh undefine <guestname>

*   A [nice quickstart](http://www.techotopia.com/index.php/Managing_Xen_using_the_xm_Command-line_Tool#Saving_and_Restoring_Xen_Guest_Systems)
    to administering Xen guests with `xm`.
*   SPICE is [supposed to be better than VNC](http://zee-nix.blogspot.com/2011/06/welcome-to-virtual-world.html)
    to remote into guests.
