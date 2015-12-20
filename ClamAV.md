Installation
------------

`   yum install clamav`  
`   freshclam`  
`   service clamd start`  
`   chkconfig clamd on`

You'll have to update only once; a `cron` job kicks in later.

Testing
-------

Send youself a plaintext message [with this
body](http://www.eicar.org/86-0-Intended-use.html):

`   X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*`

The mail log should report this (but still deliver the message)

`   amavis[14234]: (14234-03) Blocked INFECTED (Eicar-Test-Signature) `  
`   {DiscardedInbound,Quarantined}, [209.85.216.181]:48846 [209.85.216.181] `  
`   `<ping@example.com>` -> `<pong@example.com>`, quarantine: virus-1wR6qb-51v7G, `  
`   Message-ID: `<CAM=+e3EeBiwpT1eZOwB7Vt9_gK2LiMHaFtkM0w91ACePONVPmQ@mail.example.com>`, `  
`   mail_id: 1wR6qb-51v7G, Hits: -, size: 1944, dkim_sd=20120113:example.com, 590 ms`

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
