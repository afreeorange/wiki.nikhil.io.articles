This is so elegant, it makes me want to hug a penguin.

    sfdisk -d /dev/hda | sfdisk /dev/hdb

This will clone the partition map from `/dev/hda` to `/dev/hdb`. I
usually use `fdisk` (or `parted`) to clear all partitions first.
