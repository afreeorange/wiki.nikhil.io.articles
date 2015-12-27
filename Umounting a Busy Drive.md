This is all too common:

    umount: /media/backupdrive: device is busy  
    umount: /media/backupdrive: device is busy

    ## Using lsof

You could try `lsof` to see what's using the drive. Use the device name
(`/dev/sd*`) like so:

    lsof | grep sdd

### Using `fuser`

A more 'proper' approach is to use `fuser`:

    [nikhil@example ~]# fuser -m /dev/sdd5
    /dev/sdd5: 5138

So PID 5138 is using the drive. Let's see what it is:

    [nikhil@example ~]# ps aux | grep 5138
    nikhil 5138 0.2 2.7 219212 56792 ? SLl Feb11 11:25 rsync -avh --exclude-from=/home/nikhil/  anifest / /media/backupdrive

Aha! Now if you're confident that you can kill that job, go ahead and
then try umounting the drive.
