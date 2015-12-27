Quite simple, really. Use the ancient and powerful `dd` to do this!

### Backing up

    dd if=/dev/sda of=~/mbr.backup bs=512 count=1

### Restoring

    dd if=/path/to/media/mbr.backup of=/dev/sda bs=512 count=1

### Notes

The 512 byte sector is composed of three parts:

* 446 bytes for GRUB
* 64 bytes for the partition table
* 2 bytes are persistent

When restoring, these numbers become important. If restoring to a
partition of the *exact size*, use `bs=512` for backup and restore.
Else, *you have to use* `bs=446`. This is vitally important.
