Hardware
--------

Read the [Community Hardware Guide](https://forums.freenas.org/index.php?resources/hardware-recommendations-guide.12/) to pick my components this time. Cheap shit causes headaches and I expect this build to last me a while.

### Configuration

* SuperMicro X11SSM-F-O Micro-ATX: [website](https://www.supermicro.com/products/motherboard/Xeon/C236_C232/X11SSM-F.cfm), [manual](https://public.nikhil.io/X11SSM-FO.pdf)
* Intel [Xeon E3-1230 V6 Kaby Lake](https://www.newegg.com/Product/Product.aspx?Item=N82E16819117788)
* Seasonic [FOCUS Plus Series SSR-850PX 850W 80+ Platinum](https://www.newegg.com/Product/Product.aspx?Item=N82E16817151190)
* Crucial [16GB ECC Unbuffered](http://www.crucial.com/usa/en/x11ssm-f/CT7982341) memory (`CT7982341`: DDR4-2133MHz PC4-17000 ECC Unbuffered CL15 288-Pin DIMM 1.2V Dual Rank Memory Module.)
* Crucial [32GB ECC Unbuffered](https://www.newegg.com/crucial-32gb-288-pin-ddr4-sdram/p/N82E16820148900?Item=9SIAD8UB359282) memory (`CT32G4LFQ4213`: 32GB 288-Pin DDR4 SDRAM ECC DDR4-2133 PC4-17000) - later addon
* 5 x Western Digital [4TB Red NAS Drives](https://www.wdc.com/products/internal-storage/wd-red.html)
* [NZXT Source 210](http://www.amazon.com/gp/product/B005869A7K)
* [Rosewill RFT-120](http://www.newegg.com/Product/Product.aspx?Item=N82E16811988015) fan filters. Used nylon 8-32 × ½ Phillips flat-head screws to install them.
* An old Intel [80GB SSD Drive](https://www.amazon.com/Intel-SSDSA2CW080G3-Internal-Solid-State/dp/B00666SGRG)

Ran [`smartctl`](https://forums.freenas.org/index.php?threads/hard-drive-burn-in-testing-discussion-thread.21451/) short, conveyance, and long test with an ArchLinux ISO. [Two](https://www.orderfactory.com/articles/New-HDD-Testing.html) [more](https://github.com/Spearfoot/disk-burnin-and-testing/blob/master/disk-burnin.sh) resources on drive testing. Ran about 10 rounds of `memtest86` on the memory.

Software
--------

* FreeNAS 11
* Followed [this guide](https://www.youtube.com/watch?v=z2q0FYqWaVo) to install [PiHole](https://pi-hole.net/)
* [Tautulli](https://github.com/Tautulli/Tautulli-Wiki/wiki/Installation) and [Plex](http://www.freenas.org/blog/plex-on-freenas/) in separate Jails.
* Time Machine for my Mac

Notes
-----

### Node Version Manager

```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash

# Install gmake
pkg install gmake

# Export compilers
export CC=cc
export CXX=c++

# Install a version
nvm install v12.14.0
```

### Mounting NTFS Drives and Copying Things

Needed to copy some media out for mum to a Micro SD card.

```bash
# See the status of loaded modules
kldstat

# Install the FUSE package for NTFS. Seemed to ship with FreeNAS 11 so
# I didn't have to do this...
pkg install fusefs-ntfs

# Load the FUSE module
kldload fuse

# Show all the disks and their partitions.
gpart show

=>       63  970563521  da2  MBR  (463G)
         63       1985       - free -  (993K)
       2048  970561536    1  ntfs  (463G)

# The "1" refers to the partition. Make sure it's formatted NTFS
file -s /dev/da2s1

# Mount it
ntfs-3g /dev/da2s1 /mnt/windows

# Unmount it
unmount /mnt/windows
```

### `rsync` Issues

Would see this in a Jail with mounted volumes:

    rsync: mkstemp  failed: Operation not permitted

'Solved' by adding `--acls --no-perms` to the `rsync` command. Don't use `--archive` when dealing with custom ACLs.

Threads: [1](https://www.ixsystems.com/community/threads/impaired-rsync-permissions-support-for-windows-datasets.43973/), [2](https://www.ixsystems.com/community/threads/rsync-mkstemp-failed-operation-not-permitted.43269/)

### UniFi Controller

It's all here `/usr/local/share/java/unifi`
