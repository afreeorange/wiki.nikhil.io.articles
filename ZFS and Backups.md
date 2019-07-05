Backup all datasets and snapshots in a Zpool to an external USB drive.

```bash
# First, take a snapshot of the pool to back up
# I have underlying underlying datasets, so make this recursive
zfs snapshot -r source_pool@manual_backup

# Initial transfer. `pv` shows progress nicely. We
# are backing up all datasets and their snapshots.
zfs send -R source_pool@manual_backup | pv | zfs receive -vF destination_pool

# Incremental backups! First, rename the old
zfs rename -r source_pool@manual_backup source_pool@previous_backup

# Take a fresh new snapshot
zfs snapshot -r source_pool@manual_backup

# Send incrementals to the destination. The `-i` flag
# will send the difference between the two arguments to
# the destination.
#
# If all intermediary snapshots are required, use '-I'
zfs send -R -i source_pool@previous_backup source_pool@manual_backup | pv | zfs receive -v destination_pool

# Don't need the previous snapshot anymore... clean up
zfs destroy -r source_pool@previous_backup
```
