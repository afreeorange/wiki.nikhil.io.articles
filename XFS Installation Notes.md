Installed on a RHEL/CentOS 5.5, x86\_64 system.

` [root@example ~]# yum list available *xfs*`  
` Loaded plugins: fastestmirror`  
` Loading mirror speeds from cached hostfile`  
`  * addons: hpc.arc.georgetown.edu`  
`  * base: yum.singlehop.com`  
`  * epel: mirror.unl.edu`  
`  * extras: mirror.fdcservers.net`  
`  * updates: centos.mirrors.tds.net`  
` Available Packages                                         `  
` kmod-xfs.x86_64                   0.4-2                   extras`  
` kmod-xfs-xen.x86_64               0.4-2                   extras`  
` xfsdump.x86_64                    2.2.46-1.el5.centos     extras`  
` xfsprogs.x86_64                   2.9.4-1.el5.centos      extras`  
` xfsprogs-devel.x86_64             2.9.4-1.el5.centos      extras`  
` xorg-x11-xfs-utils.x86_64         1:1.0.2-4               base`

Install via:

` yum install kmod-xfs xfsdump xfsprogs xfsprogs-devel xorg-x11-xfs-utils`

Then use `parted` to create a GPT partition.

`parted /dev/sdb`  
`(parted) mklabel gpt`  
`(parted) mkpart`

Then use:

` mkfs.xfs /dev/sdb1`

Sources
-------

-   [Adding an XFS Volume in
    CentOS](http://blogwords.neologix.net/neils/?p=1)
-   [The Return of XFS on
    Linux](http://blog.2ndquadrant.com/en/2010/04/the-return-of-xfs-on-linux.html)
-   [CentOS Plus Repo and Installing
    XFS](http://wiki.centos.org/AdditionalResources/Repositories/CentOSPlus)

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
