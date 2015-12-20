Installation
------------

`   yum install clamav`  
`   freshclam`  
`   service clamd start`  
`   chkconfig clamd on`

You'll have to update only once; a `cron` job kicks in later.

Errors
------

### (!)ClamAV-clamd av-scanner FAILED:... lstat() failed: Permission denied. ERROR

Add the user `clamav` to the group `amavis`.

`   usermod --groups amavis clamav`

Then add this to `/etc/amavisd.conf`

`   AllowSupplementaryGroups yes`

This might help also

`   chmod -R 775 /var/amavis/tmp`

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
