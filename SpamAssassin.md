Installation
------------

`   yum install spamassassin`  
`   service spamassassin start`  
`   chkconfig spamassassin on`

Testing
-------

Send youself a plaintext message [with this
body](http://en.wikipedia.org/wiki/GTUBE):

`   XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X`

<http://en.wikipedia.org/wiki/GTUBE>

`   amavis[14510]: (14510-01) Passed SPAM {RelayedTaggedInbound,Quarantined}, `  
`   [209.85.216.169]:50991 [209.85.216.169] `<ping@example.com>` -> `<pong@example.com>`, `  
`   quarantine: spam-d02pBmtMd5Y9.gz, Message-ID: `<CAM=+e3GQNZPRiOXSZNFO8n6cVnbG5gGKT=NaWBM-CE3ciwomBw@mail.example.com>`, `  
`   mail_id: d02pBmtMd5Y9, Hits: 999.201, size: 1953, queued_as: 8D8297429, dkim_sd=20120113:example.com, 384 ms`

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
