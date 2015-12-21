How it works
------------

Linux uses **pages** and **dentries** to cache files and directories
(respectively) temporarily in memory to speed things up. When the
[virtual memory
subsystem](http://www.redhat.com/magazine/001nov04/features/vm/) needs
this memory for something else, the caches & dentries are flushed to a
**backing store**. In the interest of brevity, this store is a block
device, like your hard disk.

When you use either `tmpfs` or `ramfs`, *there is no backing store*.
Trippy.

### Differences between `tmpfs` or `ramfs`

Only two major things, really.

1.  **`tmpfs` can use swap** while `ramfs` cannot
2.  **`tmpfs` cannot grow dynamically** while `ramfs` can

### Why use it?

It's bloody fast. Even Karl Pilkington can tell you that.

Using `tmpfs` or `ramfs`
------------------------

### `tmpfs`

A simple tmpfs mount. This will default to half the size of system
memory:

` mount -t tmpfs tmpfs /tmp`

Mount tmpfs, but limit it to 200MB, owned by user `joe` and group
`fisherman`:

` mount -t tmpfs tmpfs /mnt/volatile -o size=200M,uid=12,gid=107`

The `man` page also says that you can tweak with block and inode counts
for your mount (`nr_blocks` and `nr_inodes`).

### `ramfs`

A simpler gentleman; no mount options whatsoever:

` mount -t ramfs ramfs /mnt/volatile`

Can you free memory after using it?
-----------------------------------

` With ramfs, there is no backing store.  Files written into ramfs allocate`  
` dentries and page cache as usual, but there's nowhere to write them to.`  
` This means the pages are never marked clean, so they can't be freed by the`  
` VM when it's looking to recycle memory.`

I'm *guessing* that this is the case with tmpfs as well. You'll probably
need to reboot the system. However, and to test this, I put two
184,320,000-byte files into `/dev/shm` and played around with them. Here
are some numbers:

` # Before `  
` MemTotal:      1026888 kB`  
` `**`MemFree:` `376068` `kB`**  
` SwapTotal:     2064376 kB`  
` SwapFree:      1891908 kB`  
` `  
` # After copying 2x180MB into 502MB tmpfs mount`  
` MemTotal:      1026888 kB`  
` `**`MemFree:` `33672` `kB`**  
` SwapTotal:     2064376 kB`  
` SwapFree:      1891908 kB`  
` `  
` # After removing one 180MB file`  
` MemTotal:      1026888 kB`  
` `**`MemFree:` `213364` `kB`**  
` SwapTotal:     2064376 kB`  
` SwapFree:      1891908 kB`

Observe that swap hasn't changed. I also couldn't get perfect arithmetic
accounting for the free memory, but it seemed close enough. Bottom-line
is that I don't know what to think of this (yet).

`/dev/shm`: A ready `tmpfs` solution
------------------------------------

Look at the output of `df -ah` on most Linux boxes. You'll see the
highlighted:

` Filesystem            Size  Used Avail Use% Mounted on`  
` /dev/mapper/VolGroup00-LogVol00`  
`                        73G  2.8G   66G   5% /`  
` proc                     0     0     0   -  /proc`  
` sysfs                    0     0     0   -  /sys`  
` devpts                   0     0     0   -  /dev/pts`  
` /dev/sda1              99M   25M   70M  26% /boot`  
` `**`tmpfs` `502M` `0` `502M` `0%` `/dev/shm`**  
` none                     0     0     0   -  /proc/sys/fs/binfmt_misc`  
` sunrpc                   0     0     0   -  /var/lib/nfs/rpc_pipefs`  
` /root/tmpfs           191M  176M   15M  93% /root/tmpfs`

`/dev/shm` (shm = shared memory) is automatically mounted to occupy half
your physical memory *at most* by default. If you're not happy with
this, go ahead and change it:

` mount -o remount,size=1G /dev/shm`

Other Points
------------

-   Don't ever think that merely *mounting* `tmpfs`, `ramfs`, or
    `/dev/shm` will actually reserve (or 'cordon off') memory.
-   In low-memory situations, swap is used as a backing store. This
    means that you're not going to see a huge performance boost with
    `tmpfs`. And I think you'd appreciate a live (albeit sluggish)
    system rather than one that's crashed due to a reckless use of
    `ramfs`. Swings and roundabouts.
