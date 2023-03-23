Backup all datasets and snapshots in a Zpool to an external USB drive. [This person](https://www.ixsystems.com/community/threads/manually-on-demand-replicating-entire-main-pool-to-external-usb-drive-for-physical-offsite-emergency.79804/) has basically the same approach except that they're doing it via manual `cron` job.

_Run all this stuff as `root`_! Get weird errors with zfs not being able to unmount before receiving snapshots :/

```bash
#!/usr/local/bin/bash

export SOURCE_POOL="source"
export DESTINATION_POOL="backup"
export SNAPSHOT_LABEL="manual_monthly_backup"
export SNAPSHOT_LABEL_PREV="${SNAPSHOT_LABEL}_previous"

# --- Initial Backup ---

# First, take a snapshot of the pool to back up
# I have underlying underlying datasets, so make this recursive
zfs snapshot -r "$SOURCE_POOL@$SNAPSHOT_LABEL"

# Initial transfer. `pv` shows progress nicely. We
# are backing up all datasets and their snapshots.
zfs send -R "$SOURCE_POOL@$SNAPSHOT_LABEL" | pv | zfs receive -vF $DESTINATION_POOL

# --- All subsequent backups ---

# Incremental backups! First, rename the old
zfs rename -r $SOURCE_POOL@$SNAPSHOT_LABEL $SOURCE_POOL@$SNAPSHOT_LABEL_PREV

# Take a fresh new snapshot
zfs snapshot -r $SOURCE_POOL@$SNAPSHOT_LABEL

# Send incrementals to the destination. The `-i` flag
# will send the difference between the two arguments to
# the destination.
#
# If all intermediary snapshots are required, use '-I'
zfs send -R -i $SOURCE_POOL@$SNAPSHOT_LABEL_PREV $SOURCE_POOL@$SNAPSHOT_LABEL | pv | zfs receive -vF $DESTINATION_POOL

# Don't need the previous snapshots anymore... clean up
zfs destroy -r $SOURCE_POOL@$SNAPSHOT_LABEL_PREV
zfs destroy -r $DESTINATION_POOL@$SNAPSHOT_LABEL_PREV
```

### `cron` Job

From that forum post up top:

```bash
(
    zfs rename -r mainpool@offsite-backup-new mainpool@offsite-backup-old;
    zfs snapshot -r mainpool@offsite-backup-new;
    zfs send -Ri mainpool@offsite-backup-old mainpool@offsite-backup-new | zfs recv -vFdu usbdrivepool;
    zfs destroy -r mainpool@offsite-backup-old;
    zfs destroy -r usbdrivepool@offsite-backup-old;
) | mail -s "FreeNAS Replication to USB Drive" "myemail@example.com"
```

### Other Notes

I ended up just making a small script that uses `rsync` and snapshots. Sample:

```bash
TIMESTAMP=$(date "+%Y-%m-%dT%H.%M.%S")

zfs snapshot "backup/miscellaneous@$TIMESTAMP"
rsync -avWHh --progress --stats /mnt/orangepool/miscellaneous/ /mnt/backup/miscellaneous/

zfs snapshot "backup/media@$TIMESTAMP"
rsync -avWHh --progress --stats /mnt/orangepool/media/ /mnt/backup/media/
```

Other useful commands:

```bash
# View snapshots for a particular pool
zfs list -t snapshot -r mypool

# View snapshot names only for a given pool (silences the header too)
zfs list -r -t snapshot -o name -H backup

# Rename a snapshot
zfs rename mypool/dataset_foo mypool/dataset_bar
```

Importing and renaming pools:

```bash
# View importable pools
zpool import

# Import a pool
zpool import my-pool

# If you messed up a pool's name, export it...
zpool export my-polo

# ... and import it with the correct name
zpool import my-polo my-pool
```

