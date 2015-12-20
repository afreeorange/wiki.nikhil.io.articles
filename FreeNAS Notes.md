\_\_NOTOC\_\_

Notes
-----

-   Drives from the same manufacturing batches are prone to
    fail together. This is bad. So purchase drives from separate
    websites/vendors to mix things up.
-   Be [careful with SATA port
    multipliers](http://www.zdnet.com/are-sata-port-multipliers-safe-7000011897).
-   Storage is cheap, and RAIDZ2 is the way to go.
-   Picking components is essentially a balancing act of these factors:
    -   Low power and/or small form-factor
    -   High capacity
    -   High redundancy
    -   High performance
    -   Low $$$
    -   Expandability
-   Used FreeNAS 8.3
-   SMB was significantly slower than AFP.
-   Wrote [a small
    script](https://github.com/afreeorange/zfs-timemachine) to snapshot
    my Macs from the FreeNAS box. Did this by creating a new account,
    setting the home folder in a ZFS filesystem, generating SSH keys,
    putting them on Macs.
    -   Runs as a cron job at midnight
    -   Script needs full paths to `zfs`, `sudo`, `rsync`, etc.

Gigabyte USB Boot Issues
------------------------

BIOS would 'forget' boot order. Configured USB stick as hard drive.
Problem solved by [upgrading from F1 to
F3](http://www.gigabyte.us/products/product-page.aspx?pid=4383&dl=1#bios).

[FreeNAS Upgrade](http://doc.freenas.org/index.php/Upgrading_FreeNAS%C2%AE)
---------------------------------------------------------------------------

[Downloaded GUI upgrade](http://www.freenas.org/download-releases.html)
for v9.2 (x86-64). Backed up config, then applied.

Got the dreaded "[Mounting failed with error
19](http://forums.freenas.org/threads/mounting-failed-with-error-19.13620/page-4)"
message no matter what I tried in BIOS (disabling XHCI, etc.) Plugging
USB stick into a 2.0 port seemed to work. ZFS volume upgrade was quick
and painless.

Hardware
--------

I didn't care about size. Also wanted a mobo that has as many onboard
SATA ports as possible. Lots of memory since ZFS [loves
memory](https://wiki.freebsd.org/ZFSTuningGuide).

| Component        | Link                                                                                | Price   |
|------------------|-------------------------------------------------------------------------------------|---------|
| Case             | [NZXT Source 210](http://www.amazon.com/gp/product/B009O7YZ7O)                      | $79.99  |
| PSU              | [Rosewill Capstone 450W](http://www.amazon.com/gp/product/B006BCKDGW)               | $59.99  |
| Motherboard      | [Gigabyte GA-F2A85XM-D3H](http://www.amazon.com/gp/product/B005869A7K)              | $39.99  |
| CPU              | [AMD A4-5300 APU 3.4Ghz](http://www.amazon.com/gp/product/B0095VPBM2)               | $49.99  |
| Memory           | [Corsair Vengeance 16GB](http://www.amazon.com/gp/product/B006EWUO22)               | $129.99 |
| Storage (x4)     | [Seagate Barracuda 3TB SATA 6 Gb/s](http://www.amazon.com/gp/product/B005T3GRLY)    | $120    |
| FreeNAS stick    | [ADATA S102 8GB USB 3.0](http://www.amazon.com/gp/product/B005Y8BYOE)               | $20     |
| Fan Filters (x4) | [Rosewill RFT-120](http://www.newegg.com/Product/Product.aspx?Item=N82E16811988015) | $4      |

-   Used nylon 8-32 × ½ Phillips flat-head screws for the fan-filters.

Software
--------

Need to have NOPASSWD sudo access for my [backup
scripts](https://github.com/afreeorange/zfs-timemachine). Created a user
in the "wheel" group and added a cron job via the web interface. Bad
part is that I have to do this with every update.

    su -

    # Mount read-write
    mount -wu /

    # Edit sudoers file
    chmod u+w /conf/base/etc/local/sudoers
    echo -e "# For ZFS snapshots\n%wheel ALL=(ALL) NOPASSWD: ALL" >> /conf/base/etc/local/sudoers
    chmod u-w /conf/base/etc/local/sudoers

    # Mount read-only
    mount -ru /

References
----------

-   <http://blog.brianmoses.net/2013/01/diy-nas-2013-edition.html>

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")
[Category: FreeNAS](Category:_FreeNAS "wikilink")
