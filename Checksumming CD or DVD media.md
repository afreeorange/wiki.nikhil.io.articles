**Note on the MD5 command**: On OS X, it's `md5`. On RHEL it's `md5sum`.
I'll use the latter for this guide.

### Doing it properly

-   For this demo, I'll attempt to burn a CentOS 5.5 DVD
-   The ISO name is `centos-5.5.i386.iso`
-   It was burned to a DVD drive with device name `/dev/sdc`

Checksumming an ISO image is easy:

` [user@localhost ~]# md5sum centos-5.5.i386.iso`  
` 75c92246479df172de41b14c9b966344`

When you burn this to a DVD, you can verify that everything's good by
first getting the size of the ISO in bytes:

` [user@localhost ~]#  ls -l centos-5.5.i386.iso`  
` -rw-r--r-- 1 user user 4185118720 May 20 15:40 centos-5.5.i386.iso`

Now divide that by 2048 (4185118720 ÷ 2048 = 2043515). You can now
verify the DVD the ISO was burned to using:

` [user@localhost ~]# dd if=/dev/sdc `**`bs=2048`**` `**`count=2043515`**` | md5sum`  
` 75c92246479df172de41b14c9b966344`

This verifies a successful burn. Merely using `dd if=/dev/sdc | md5sum`
will *always* give you an incorrect checksum!

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
