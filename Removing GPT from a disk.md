The GUID Partition Table (GPT) is unique in that it resides both at the
beginning and the end of a disk. Its removal is not as simple as zeroing
out the first 512 (or 446) bytes.

For demonstration, I have a Seagate 750GB drive with a device name `sdb`

Figure out where to start zeroing

` [root@livecd centos]# fdisk -s /dev/sdb`  
` 732574584`

This is the total number of blocks; we'll zero out the last 100,000
blocks:

` dd if=/dev/zero of=/dev/sdb seek=732500000 bs=1k`

Now zero out the first 1000 (overkill, really)

` dd if=/dev/zero of=/dev/sdb count=1000 bs=1k`

Ta da!

Sources
-------

-   [GPT Removal](http://www.digital52.com/help/gptremoval.html)
