Installation
------------

If your processor supports vmx (`cat /proc/cpuinfo | grep vmx`), you can
install QEMU/KVM for faster VMs.

` yum install kvm kmod-kvm qemu libvirt`  
` service libvirtd start`

You can now connect to the instance with
[`virt-manager`](http://virt-manager.org/).

Errors
------

If virt-manager reports that you don't have KVM support, check your BIOS
and enable virtualization (VT for Intel processors) and power cycle the
server. Then check if the kernel module is inserted properly:

` lsmod | grep kvm`  
` `  
` # If not`  
` modprobe kvm`  
` modprobe kvm-intel`

If the last commands don't work, *make sure your kernel isn't Xen!*. I
made this mistake and switching to a non-Xen kernel ensured that the KVM
kernel module was inserted properly.

### could not query memory balloon allocation

You can only give a VM 4GiB of memory under QEMU

### MP-BIOS bug: 8254 timer not connected to IO\_APIC

Add a `noapic` to your kernel boot params.

Sources
-------

-   <http://itscblog.tamu.edu/startup-guide-for-kvm-on-centos-6/>
-   <http://eduardo-lago.blogspot.com/2012/01/how-to-install-kvm-and-libvirt-on.html>



