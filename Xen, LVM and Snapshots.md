References
----------

-   A [most excellent explanation of using snapshots as
    backups](http://serverfault.com/q/24126)
-   [Live Xen Snapshotting](http://serverfault.com/q/71909) strategy
-   [xenBackup](https://github.com/doofdoofsf/xenBackup) - A script to
    automate backups using tar, rsync or rdiff-backup

### \[<http://searchservervirtualization.techtarget.com/tip/Creating-snapshots-in-Xen-with-Linux-commands?ShortReg=1&mboxConv=searchServerVirtualization_RegActivate_Submit>& Archived Article\]

A virtual machine snapshot is a great feature, freezing the current
state of a virtual machine. Unfortunately, open source Xen doesn't offer
support for snapshots -- but Linux does. Since open source Xen always
uses Linux as its privileged domain, you can use Linux commands to
create snapshots.

**Byte-by-byte snapshot**  
 One way of making snapshots in Xen is by using Linux `dd` after saving
the current state of a virtual machine. This would involve the following
steps:

1.  Use the `xm save` command to disable the current state of a virtual
    machine and write it to a disk file. This would write the machine
    state only to a file, not the current state that is used in the Xen
    disk files or partitions. To do this for a domain with the name
    linux01, use `xm save linux01 linux01.sav`. Take note that this
    command stops the virtual machine.
2.  Now dump the current state of the disk image files to a backup file
    using `dd`. The following example would do this for LVM logical
    volumes used by Xen:  
    `dd if=/dev/xenvols/linux01_root of=/data/xen_linux01_root.img`  
3.  Restart the virtual machine using the `xm` restore command.

The major disadvantage of this solution is time. The `dd` command makes
a byte-by-byte copy of the virtual machine disk file and that can take
an incredibly long time. Therefore, this option may not be very
practical.

**The LVM method**  
 In Linux, the Logical Volume Manager (LVM) can also be used for
creating a snapshot, one that takes significantly less time than the
previous disk file method. This method implies that your virtual machine
uses an LVM logical volume as its storage back-end, as opposed to using
a virtual disk file. For this logical volume, you next need to create a
snapshot. This snapshot is a kind of backup that contains metadata and
blocks that have changed from the moment that you took the snapshot
only. The trick is that when you use `dd` to make a copy of the snapshot
via the metadata, you'll always make a snapshot of the original blocks
on the original volume without the need to de-activate the original
volume as well. By working this way, you can reduce the time it takes to
create a snapshot of the virtual machine drastically. The procedure is
as follows:

1.  Use the xm save command to save the current state of the virtual
    machine and write it to a disk file:  
    `xm save linux01 linux01.sav`
2.  Assuming that you already have an LVM logical volume that is used as
    the storage back-end for your virtual machine, use the following
    command to make a snapshot of this volume. A good guideline is to
    use 10% of the allocated disk space in the original logical volume
    as the size to use for the snapshot volume:  
    `lvcreate -s -L 1G -n linux01-snap /dev/xenvols/linux01`
3.  Since you've now saved the state of your virtual machine in the LVM
    snapshot, you can restart the virtual machine, reducing the down
    time of the virtual machine dramatically as compared to the method
    sketched above:  
    `xm restore linux01-sav`
4.  Use `dd` to create the snapshot of the virtual machine and write it
    to an image file. This will take longer since by using the snapshot
    you will copy all the disk blocks that are allocated by the virtual
    machine:  
    `dd if=/dev/xenvols/linux01-snap of=/data/xen01.img`
5.  Don't forget to remove the snapshot in the last step of
    this procedure. This is important because a snapshot that stays
    around will eventually fill up completely and when that happens the
    snapshot will be disabled. The problem with this is that it will
    prevent you from remounting the original volume as well, so don't
    forget to apply this last step:  
    `lvremove /dev/xenvols/linux01-snap`

While no Linux distribution offers a solution in the open source Xen
stack to create a virtual machine snapshot as of yet, you've read how
you can do it anyway by using standard Linux tools such as LVM and `dd`.

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
