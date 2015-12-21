The chief difference between defining a mount in `fstab` versus `autofs`
is that the kernel dedicates resources to keep `fstab` mounts in place
all the time. Conversely, `autofs` mounts are "on-demand" and are better
suited for NFS or DVD drives.

Understanding the `fstab`
-------------------------

Here's a sample `fstab`:

` # Sample fstab`  
` # default = rw, suid, dev, exec, auto, nouser, and async`  
` `  
` # Device                  Mount Point   FS Type  Options         Dump?   fsck`  
`                                                `  
` /dev/VolGroup00/LogVol00  /             ext3     defaults        1       1`  
` LABEL=/boot               /boot         ext3     defaults        1       2`  
` tmpfs                     /dev/shm      tmpfs    defaults        0       0`  
` devpts                    /dev/pts      devpts   gid=5,mode=620  0       0`  
` sysfs                     /sys          sysfs    defaults        0       0`  
` proc                      /proc         proc     defaults        0       0`  
` /dev/VolGroup00/LogVol01  swap          swap     defaults        0       0`

| Column                   | What it is                                                                                                                                                                                                                                                                                  |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Device**               | Name of the block device (like `/dev/sdc1`, etc.) You can also use `LABEL=` or `UUID=`                                                                                                                                                                                                      |
| **Mount Point**          | Where to mount the device                                                                                                                                                                                                                                                                   |
| **FS (FileSystem) Type** | Can be `ext2`, `ext3`, `reiserfs`, `vfat` and `swap` (many others depending upon kernel support.) The interesting one is `auto`, whereby the kernel attempts to detect the filesystem automagically. This is good for CD/DVD-ROMs and (gasp!) floppies. Use an actual FStype for all others |
| **Options**              | For a much more comprehensive list of options, see `man mount`. Here are some common ones:                                                                                                                                                                                                  
                            `auto`/`noauto` - Mount/do not mount at boot                                                                                                                                                                                                                                                 
                            `user`/`nouser` - Should normal users be able to mount? If so, use **user**. Else, it's **nouser**                                                                                                                                                                                           
                            `exec`/`noexec` - Execute binaries off the device? **noexec** off a Linux root partition might be unpleasant...                                                                                                                                                                              
                            `ro` - Mount as read-only                                                                                                                                                                                                                                                                    
                            `rw` - Mount as read/write                                                                                                                                                                                                                                                                   
                            `sync`/`nosync` - Changes to device are done synchronously with **sync** (written at the same time the command is issued). Good for USB sticks and (gasp!) floppies. **nosync** may be preferable for HDD performance.                                                                       |
| **Dump**                 | This is a small backup utility. Set to `0` almost all the time.                                                                                                                                                                                                                             |
| **fstab**                | Determine the order in which to check filesystems on devices. `0` will cause it to ignore it.                                                                                                                                                                                               |

Using better device identifiers
-------------------------------

Given that the device names might change due to myriad factors (e.g.
plug HDD into a different USB or SATA port), it's a better idea to use
the device label or, even better, the device's UUID. I'm going to deal
with UUIDs in this example.

### Figure out the device's UUID

UNIX-like simplicity itself.

` [user@example ~]# `**`ls` `-l` `/dev/disk/by-uuid/`**  
` total 0`  
` lrwxrwxrwx 1 user user 10 Jun 25 11:00 43b64bd4-ee2b-4e4d-b2af-e868cb47d596 -> ../../sda1`  
` lrwxrwxrwx 1 user user 10 Jun 25 11:00 7c488bee-a567-4432-bdbe-c30e9d33a927 -> ../../sdb5`  
` lrwxrwxrwx 1 user user 10 Jun 25 11:00 c1e9157e-6572-4ec1-b1d0-8adb97052a59 -> ../../sdd1`  
` lrwxrwxrwx 1 user user 10 Jun 28 10:25 edabbb0e-d87a-4c5b-a9aa-492e553c98d1 -> ../../sdc5`

You can see what each device is using `fdisk -l /dev/sdc5` or, since you
know the UUID:

` [user@example ~]# `**`fdisk` `-l`
`/dev/disk/by-uuid/edabbb0e-d87a-4c5b-a9aa-492e553c98d1`**  
` `  
` Disk /dev/disk/by-uuid/edabbb0e-d87a-4c5b-a9aa-492e553c98d1: 1000.2 GB, 1000202208768 bytes`  
` 255 heads, 63 sectors/track, 121600 cylinders`  
` Units = cylinders of 16065 * 512 = 8225280 bytes`

### Create the `fstab` entry

` UUID=edabbb0e-d87a-4c5b-a9aa-492e553c98d1 /media/usbdrive ext3  defaults  0 0`

Et voila!

### Problems with this approach

You might see a **special device
UUID=edabbb0e-d87a-4c5b-a9aa-492e553c98d1 does not exist** when you
reboot your system, especially when dealing with external drives. Some
people report that specifying the full path helps (e.g.
UUID=/dev/disk/by-uuid/...) but I haven't had any luck with this.

Essentially, and to keep things simple, you could still keep the `fstab`
entry and issue a `mount -a` to mount all the devices you've specified
*after* booting up. While there's really no 'fix' to this AFAIK (even
using `LABEL=` doesn't help), you could alternately add a quick line to
`/etc/rc.local`.

` echo -e "Waiting for external drives to mount..." && sleep 30 && echo -e "Done waiting."`

This *has* worked for me, even though I see the "special device" error
at startup.

Miscellaneous
-------------

-   `blkid` is another way to figure out UUIDs.

` [user@example ~]# `**`blkid`**  
` /dev/mapper/VolGroup00-LogVol01: TYPE="swap" `  
` /dev/mapper/VolGroup00-LogVol00: UUID="5aafa32b-d5c0-40a7-9a6a-417b8a9c7832" TYPE="ext3" `  
` /dev/sda1: LABEL="/boot" UUID="43b64bd4-ee2b-4e4d-b2af-e868cb47d596" TYPE="ext3" `  
` /dev/hda: LABEL="CentOS_5.3_Final" TYPE="iso9660" `  
` /dev/VolGroup00/LogVol00: UUID="5aafa32b-d5c0-40a7-9a6a-417b8a9c7832" TYPE="ext3" `  
` /dev/VolGroup00/LogVol01: TYPE="swap" `  
` /dev/sdc5: UUID="edabbb0e-d87a-4c5b-a9aa-492e553c98d1" TYPE="ext3" SEC_TYPE="ext2" `  
` /dev/sdb5: LABEL="/rsync_drive_3" UUID="7c488bee-a567-4432-bdbe-c30e9d33a927" TYPE="ext3" `  
` /dev/sdd1: UUID="c1e9157e-6572-4ec1-b1d0-8adb97052a59" TYPE="ext3"`

-   To see the changes you've made to `fstab` without rebooting your
    system, use `mount -a`

<!-- -->

-   Samba shares with spaces in them can get a little tricky. I've tried
    single, double quotes with no success. Octal representation of a
    space (`\040`) seemed to work :) Here's an example:

` //19.67.18.160/Lab`**`\040`**`Documents /media/secretary cifs rw,credentials=/media/secretary.credentials 0 0`

I've stored the credentials for the connection in the file
`/media/secretary.credentials`. It has `600` permissions and looks like
this:

` username=nikhil`  
` password=OMG_FLUFFY_PUPPIES`



