Pre-Flight
----------

-   Installed on CentOS 6.2 x86\_64 running on KVM.
-   Decided to use LVM to manage the partition I wanted to encrypt for
    future expansion.

### Install the necessary tools

If you did a 'minimal' CentOS 6.x install, you'll need these:

` yum install cryptsetup device-mapper util-linux`  
` modprobe dm_crypt`  
` lsmod | grep dm_crypt`

Prepare the Device
------------------

I used LVM. This section could've been about making a software RAID. If
you've prepared your device or have a standard disk (e.g. `/dev/sdb1`),
you can skip to the next section.

` pvcreate /dev/vda2`  
` vgcreate volgroups /dev/vda2`  
` lvcreate -l 100%FREE -n secure volgroups`

You now have a block storage device at `/dev/mapper/volgroups-secure`.
You'll create an encrypted device using it.

Creating the Encrypted Device
-----------------------------

` cryptsetup luksFormat /dev/mapper/volgroups-secure`  
` cryptsetup luksOpen   /dev/mapper/volgroups-secure secure`

This creates the device `/dev/mapper/secure`. The cipher used is
AES-256-CBC. Fill it with junk; will take time, but this will prevent
people from knowing the size of data on your device.

` dd if=/dev/urandom of=/dev/mapper/secure`

Now create a filesystem

` mkfs -t ext4 /dev/mapper/secure`

Mount it!

` mount -t ext4 /dev/mapper/secure /mnt/secure`

Close it when done:

` crypsetup luksClose secure`

Mounting at boot
----------------

-   Add this to `/etc/crypttab` (create with 0644 if it doesn't exist)

` secure   /dev/mapper/volgroups-secure`

-   Then add this to `/etc/fstab`

` /dev/mapper/secure        /mnt/secure                   ext3    defaults        0 0`

LVM Resizing
------------

For the example above,

` lvextend -L+2048G /dev/mapper/volgroups-secure`  
` resize2fs /dev/mapper/secure`

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
