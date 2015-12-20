Using `parted`
--------------

Here's a transcript from formatting a WD20EARS drive on an x86\_64,
RHEL5.5, stock box. **Setting the unit format to sectors is bloody
important!**

` [root@example ~]# parted /dev/sdb`  
` GNU Parted 1.8.1`  
` Using /dev/sdb`  
` Welcome to GNU Parted! Type 'help' to view a list of commands.`  
` (parted) print                                                            `  
` `  
` Model: ATA WDC WD20EARS-00M (scsi)`  
` Disk /dev/sdb: 2000GB`  
` Sector size (logical/physical): 512B/512B`  
` Partition Table: msdos`  
` `  
` Number  Start  End  Size  Type  File system  Flags`  
` `  
` `**`(parted)` `unit` `s`**  
` (parted) mkpart`  
` Partition type?  primary/extended? primary                                `  
` File system type?  [ext2]? `  
` Start? 64                                                                 `  
` End? -1                                                                   `  
` (parted) print                                                            `  
` `  
` Model: ATA WDC WD20EARS-00M (scsi)`  
` Disk /dev/sdb: 2000GB`  
` Sector size (logical/physical): 512B/512B`  
` Partition Table: msdos`  
` `  
` Number  Start  End          Size         Type     File system  Flags`  
`  1      64s    3907029167s  3907029104s  primary`  
` `  
` (parted) quit                                                             `  
` Information: Don't forget to update /etc/fstab, if necessary.`

Using `fdisk`
-------------

I haven't tried this but here's the command with flags that will align
the partition to 4K boundaries:

`  fdisk -H 224 -S 56 /dev/sdb`

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
