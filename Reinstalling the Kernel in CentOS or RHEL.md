* `yum` does not remove the kernel entirely.
* You can remove and reinstall the kernel since it's entirely in
    memory

### Reinstalling your kernel

To see a list of installed kernels:

    [root@server ~]# rpm -qa | grep kernel | sort  
    kernel-2.6.18-308.11.1.el5  
    kernel-2.6.18-308.13.1.el5  
    kernel-2.6.18-308.4.1.el5  
    kernel-2.6.18-308.8.1.el5  
    kernel-2.6.18-308.8.2.el5  
    kernel-headers-2.6.18-308.13.1.el5  
    kernel-xen-2.6.18-308.11.1.el5  
    kernel-xen-2.6.18-308.13.1.el5  
    kernel-xen-2.6.18-308.4.1.el5  
    kernel-xen-2.6.18-308.8.1.el5  
    kernel-xen-2.6.18-308.8.2.el5

To remove specific ones,

    yum remove kernel-2.6.18-308.11.1.el5 kernel-2.6.18-308.13.1.el5

And so on. Now you can

    # Install the version you want   
    yum install kernel-2.6.18-308.13.1.el5  
      
    # or the latest version  
    yum install kernel

Check `/boot/grub/menu.lst`, and then reboot.
