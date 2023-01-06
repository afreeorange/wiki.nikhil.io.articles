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
