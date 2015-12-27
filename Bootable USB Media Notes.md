Assume that your USB disk's drive letter is **D:**

CentOS/Fedora Bootable Installation Key
---------------------------------------

* Clear all partitions with `fdisk`
* Create a 100M primary, bootable ('a' key), FAT ('t' and then 'b')
    partition
* Then create another primary partition to fill up the rest of the
    space
* Write changes and then issue:

        mkfs -t vfat /dev/<FAT Partition>  
        mkfs /dev/<Linux Partition>

* Install `livecd-tools`
* Run it:

        livecd-iso-to-disk /full/path/to/CentOS_image.iso /dev/<FAT Partition>

* Mount the *Linux partition* and copy the ISO file
* You're set. [Thanks, brah](http://lists.centos.org/pipermail/centos/2010-April/093806.html).

On Windows
----------

### Step 1 : Use `diskpart` to create bootable media

    diskpart   
    select disk 1
    clean   
    create partition primary   
    select partition 1   
    active   
    format fs=fat32   
    assign   
    exit

For the step highlighted above, use `list disk` to make sure that your
USB stick is, indeed, **disk 1**. At this point, the USB stick has a
primary partition and should be bootable. However, it doesn't have
anything to *boot* per se.

### Step 2 : Use `xcopy` to copy over the files to be booted

From the directory containing the boot files (these could be Windows
installation files, for example), issue:

    xcopy *.* /s/e/f D:\

Where **D:** is the drive letter of your USB stick. Et voila! Install
away!

Other notes and resources
-------------------------

* [Unetbootin](http://unetbootin.sourceforge.net/) is a great tool for
    bootable USB on Windows and Linux (GUI only)
    * The [LiveUSB Creator](https://fedorahosted.org/liveusb-creator/)
      is another option.
* [Hiren's Boot CD](http://www.hirensbootcd.net) remains my favorite
    tool for all sorts of diagnostics
    * Also has a [dandy USB formatting
      tool](http://www.hirensbootcd.net/usb-booting.html)
* Linux distros [like Knoppix](http://www.knoppix.org) might also be
    useful for this purpose
    * [Other elaborate
        resources](http://wiki.linuxquestions.org/wiki/Booting_from_USB)
        also exist!
* An excellent idea would be to [roll everything into one using
    GRUB](http://www.neowin.net/forum/index.php?showtopic=621496)

