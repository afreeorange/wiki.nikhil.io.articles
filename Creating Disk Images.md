Floppy/CD/DVD drives
--------------------

### Creating ISOs

The venerable `dd` is fantastic for this. Let's say your device is
`/dev/cdrom`:

` dd if=/dev/cdrom of=new_cdrom.iso`

This will create an ISO9660-compliant image.

### Mounting ISOs

ISO files can be [loop
mounted](http://en.wikipedia.org/wiki/Loop_device) like so:

` mount -o loop new_cdrom.iso /mount/point`

A random bunch of files/directories
-----------------------------------

For this example, I want to create an ISO9660-compliant image of a
directory, `/home/nikhil/Documents`:

` mkisofs -o /home/nikhil/`**`Documents_backup.iso`**` /home/nikhil/Documents`

<font color="red">**Important**</font>: `mkisofs` can only create ISOs
from directory trees that are **a maximum of 6 levels deep**! Any more
and it will croak.

Hard Drives
-----------

This is quite tricky. You might want to consider
[Mondo](http://www.mondorescue.org/) for its awesomeness. While it
*does* create ISOs, their contents are compressed and divvied up into
many separate files. If you want an 'image' that you can carry around
and mount, read on.

### Using `dd`

Let's say your HDD is labeled `/dev/sdb`. You can use `dd` to create a
block-by-block replica image of this drive like so:

` dd if=/dev/sdb of=hard_drive_backup.img conv=sync,noerror`

Simple, eh? What's tricky is that `dd` will go through **all** blocks,
even the unallocated ones. This means that if you have a 300GB drive
with just 10GB of data, you're going to get an image that's 300GB in
size (can you see why Mondo or Ghost make more sense?)

The `conv=sync,noerror` will tell `dd` to write 'something' to the image
in case it encounters a bad block.

#### Speeding up the process

We could use a bigger block size. Using the default 512-byte block size
can slow things down considerably.

` dd if=/dev/sdb of=hard_drive_backup.img conv=sync,noerror `**`bs=64K`**

#### Making the process space-efficient

We could *also* use good 'ol `gzip` or `bzip2` like so:

` dd if=/dev/sdb conv=sync,noerror bs=64K `**`|` `gzip` `-c` `>`
`hard_drive_backup.img.gz`**

### Mounting a `dd` image

The resultant image from the previous steps is absolutely *not*
ISO9660-compliant. Any attempts to mount it as such might result in mild
embarrassment. To use the image, try `fdisk`:

` [user@localhost mondo_archives]# fdisk -ul hard_drive_backup.img`  
` last_lba(): I don't know how to handle files with mode 81a4`  
` You must set cylinders.`  
` You can do this from the extra functions menu.`  
` `  
` Disk hard_drive_backup.img: 0 MB, 0 bytes`  
` 255 heads, 63 sectors/track, 0 cylinders, total 0 sectors`  
` Units = sectors of 1 * 512 = `**`512` `bytes`**  
` `  
`             Device Boot           Start         End      Blocks   Id  System`  
` hard_drive_backup.disk1   *          63      208844      104391   83  Linux`  
` hard_drive_backup.disk2          208845     2313359     1052257+  82  Linux swap / Solaris`  
` `**`hard_drive_backup.disk3` *`2313360`* `78156224` `37921432+` `83`
`Linux`**  
` Partition 3 has different physical/logical endings:`  
`     phys=(1023, 254, 63) logical=(4864, 254, 63)`

#### The tricky part

To mount the third partition (you know, the one with all the data on
it), you need some quick arithmetic: *Multiply the starting cylinder by
the block size and use it as the offset for the `mount` command.*

**For example**: In the output above, the offset would be
`512 x 2313360 = 1184440320`. You would then use this value like so:

` mount -o loop,`**`offset=1184440320`**` hard_drive_backup.img /mount/point`



