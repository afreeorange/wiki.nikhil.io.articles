Didn't work for me when backing up to a TM volume on FreeNAS, but oh well.

```bash
#! /usr/bin/env bash
#
# Source:
# https://www.reddit.com/r/freenas/comments/akqd2d/time_machine_must_create_a_new_backup_for_you/
#
set -euo pipefail
IFS=$'\n\t'

if [[ $(whoami) != 'root' ]]; then
    exit 1
fi

read -r -p 'Enter Time Machine Hostname: ' HOSTNAME
read -r -p 'Enter Share: ' SHARE
read -r -p 'Enter Username: ' USERNAME
read -r -s -p 'Enter Password: ' PASSWORD

TM_NAME=$(hostname -s | sed -e 's/-/ /g')
MOUNT=/Volumes/TimeMachine
SPARSEBUNDLE=$MOUNT/$TM_NAME.sparsebundle
PLIST=$SPARSEBUNDLE/com.apple.TimeMachine.MachineID.plist

echo "Disabling Time Machine"
tmutil disable

echo "Mounting volume"
mkdir -p $MOUNT
mount_afp "afp://$USERNAME:$PASSWORD@$HOSTNAME/$SHARE" "$MOUNT"

# If mounting via SMB
# mount_smbfs "smb://$USERNAME:$PASSWORD@$HOSTNAME/$SHARE" "$MOUNT"

echo "Changing file and folder flags"
chflags -R nouchg "$SPARSEBUNDLE"

echo "Attaching sparse bundle"
DISK="$(hdiutil attach -nomount -readwrite -noverify -noautofsck "${SPARSEBUNDLE}" | grep Apple_HFS | cut -f 1 | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//')"
echo "$DISK"
read -r -p "Press enter to continue"

echo "Repairing volume"
#diskutil repairVolume $DISK
fsck_hfs -fry $DISK

echo "Fixing Properties"
cp "$PLIST" "$PLIST.backup"
sed -e '/RecoveryBackupDeclinedDate/{N;d;}'   \
-e '/VerificationState/{n;s/2/0/;}'       \
"$PLIST.backup" \
> "$PLIST"

echo "Unmounting volumes"
hdiutil detach "$DISK"
umount "$MOUNT"

echo "Enabling Time Machine"
tmutil enable

echo "Starting backup"
tmutil startbackup

exit 0
```
