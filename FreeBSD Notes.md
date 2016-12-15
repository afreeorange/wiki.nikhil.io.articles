[TOC]

Miscellaneous
-------------

### System Information

You'll find everything you need in `/var/run/dmesg.boot` or in the
output of `sysctl -a`. Some examples:

```bash
# Memory  
grep memory /var/run/dmesg.boot  
  
# No. of CPUs  
grep CPU /var/run/dmesg.boot
```

### Kernel Modules

`*kldstat*` shows all loaded modules. `*kldload*` loads a
module, `*kldunload*` unloads a module.

### [Firewall](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/firewalls-ipfw.html)

I used the venerable IPFW. In `/etc/rc.conf` enable the firewall and
provide a script:

    firewall_enable="YES"  
    firewall_script="/opt/firewall"

Then reboot. I used my own simple script at
`git://git.example.com/it.firewall.ipfw`. However, given that this is
exactly what OS X 10.6 (and below) use for a firewall, you'll find
[either WaterRoof or NoobProof](http://www.hanynet.com/comparison.html)
pretty awesome for some complicated rules. You'll have to modify their
outputs to script format.

### Listing Disks

Can do one of the following

    # For SCSI disks [1]
    cat /var/run/dmesg.boot | grep da
      
    # or  
    gpart show

### SNMP

You'll need to [install
bsnmp](http://community.zenoss.org/docs/DOC-9132) to get this working.

### [Mounting XFS](http://www.freebsd.org/doc/handbook/filesystems-linux.html)

    kldload xfs  
    mount -t xfs -o ro /dev/da15p1 /mnt

If you get an "Operation not permitted" error, check your filesystem.
For XFS, these packages might be helpful

    pkg_add -r xfs xfsprogs xfsinfo

### The backspace 'problem' in `vim`

    echo 'set backspace+=start,eol,indent' >> ~/.vimrc

### A note on `/etc/rc.conf`

If you screw up this file, you're screwed in turn. A single missing
quote and you'll boot into a read-only system. Be careful, and use
`sysinstall` if you can.

The Ports Tree
--------------

See the ["Obtaining the Ports
Collection"](http://www.freebsd.org/doc/handbook/ports-using.html)
section of the manual if you forgot to install the tree. To update, just
run `portsnap update`.

### Installing stuff

As root,

```bash
# Update ports tree and system packages
portsnap fetch extract update
freebsd-update fetch
freebsd-update install

# This installs pkg
pkg install bash \
            bash-completion \
            chromium \
            inconsolata-ttf \
            numix-theme
            rsync \
            sudo \
            tree \
            vim \
            xfce \
            xorg \
            git \

# Get the kernel source for VirtualBox Guest Additions if applicable
# Configure proxy in ~/.subversion/servers if needed
svnlite checkout http://svn.freebsd.org/base/head /usr/src
svnlite up /usr/src
cd /usr/src
make clean
```

#### No packages matching xxx available in the repository

    rm /var/db/pkg/repo-*
    pkg upgrade

### Installing `mkfile`

Quicker alternative to ye olde `dd`:

    cd /usr/ports/sysutils/mkfile; make install clean

### Installing Binaries

Use [the packages system](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/packages-using.html).

    pkg_add -r vim

### [Searching ports](http://www.freebsd.org/ports/searching.html)

    cd /usr/ports  
    make search name=<package>

NFS Exports
-----------

Here's [the pertinent
page](http://www.freebsd.org/doc/handbook/network-nfs.html) from the
FreeBSD manual that describes how to set these up.

*   Make sure that you issue `/etc/rc.d/mountd onereload` *and* observe
    the output of `/var/log/messages` for any errors.
*   You should then verify your mounts by issuing `showmount -e`

ZFS
---

You will need to increase `kmem` to prevent panics. Check `vm.kmem_size`
and `vm.kmem_size_max` using:

    sysctl -a

If the values look appropriate, skip [the "Loader
Tunables"](http://www.freebsd.org/doc/handbook/filesystems-zfs.html#AEN27881)
section of the manual.

Now start ZFS with:

    echo 'zfs_enable="YES"' >> /etc/rc.conf  
    /etc/rc.d/zfs start

### Creating a zpool

I had twelve 3TB disks I wanted in a RAID6. Sun calls it "RAIDZ2", since
it averts the [RAID write
hole](http://en.wikipedia.org/wiki/RAID_5_write_hole). I wanted to call
my pool "**data**":

    zpool create data raidz2 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11 da12

I also had a 256GB solid-state cache drive to speed up performance using
[ARC](http://en.wikipedia.org/wiki/Adaptive_replacement_cache). Its
device name is `da16`

    zpool add data cache da16

Finally, [the ZFS tuning guide](http://wiki.freebsd.org/ZFSTuningGuide)
is a must-read. There's also a longer, [evil
version](http://www.solarisinternals.com/wiki/index.php/ZFS_Evil_Tuning_Guide).

### Using an existing pool

I had to reinstall FreeBSD after some XFS weirdness. I was able to get
my old pool back with

    zpool import

This scans all drives and lists any available pools. Then actually
import the pool with

    zpool import data

### [Migrating ZFS pools](http://docs.oracle.com/cd/E19963-01/html/821-1448/gbchy.html)

    # On the old server, export the pool ("zpool export -f data" to force)  
    zpool export data  
      
    # You've now moved the disks to a new server and are on it right now  
      
    # Scan for zpools  
    zpool import  
      
    # Import the zpool (use "-f" if necessary)  
    zpool import -f data

It's magical: zpool will also tell you the host that the drives were on,
which ones are missing, etc.

### Creating Filesystems

It's possible to mount and use zpools. But you will miss out on awesome
things that ZFS is known for. So create a filesystem:

    zfs create data/users  
  
    # Set compression on  
    zfs set compression=gzip data/users

You may want to think twice before you set compression and
deduplication. They take a heavy toll on memory and performance. Here
are some resources:

*   [Dedup or not dedup](http://constantin.glez.de/blog/2011/07/zfs-dedupe-or-not-dedupe)
*   [Dedup versus compression](http://www.edugeek.net/forums/nix/49844-zfs-deduplication-dedup-vs-compression.html)
*   [Oracle blog post on testing compression](https://blogs.oracle.com/observatory/entry/zfs_compression_a_win_win)

### [Snapshots and Clones](http://blog.allanglesit.com/2011/04/zfs-snapshot-management/)

Using `date +%Y-%m-%dT%k:%M:%S` for a nice ISO-8601 formatted date:

    # Take a snapshot  
    zfs snapshot tank/data@2012-09-09T18:06:49  
  
    # DIsplay snapshots  
    zfs list -t snapshot

### NFS

ZFS is awesome with NFS. Here, I share a filesystem with my trusted
network, after setting up my NFS server according [to the FreeBSD
manual](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-nfs.html).

    zfs sharenfs='-network 10.212.8.0 -mask 255.255.255.0 -maproot root' tank/home

Export data is *not* written to `/etc/exports` (as you would expect) but
to `/etc/zfs/exports`. Check your mounts with the usual:

    [root@lucifer ~]# showmount -e  
    Exports list on localhost:  
    /tank/home                         10.212.8.0

Unlike traditional NFS, you don't have to restart `mountd` every time
you redefine your mount points.

Here are further resources:

*   [ZFS and NFS on FreeBSD](http://misc.allbsd.de/Vortrag/EuroBSDCon_2007//Pawel_Jakub_Dawidek/eurobsdcon07_zfs.pdf)
*   [NFS section of the FreeBSD manual](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-nfs.html)

### Other Resources

*   [ZFS Best Practices Guide](http://www.solarisinternals.com/wiki/index.php/ZFS_Best_Practices_Guide)

[^1]: (http://www.freebsd.org/doc/handbook/disks-naming.html)  
