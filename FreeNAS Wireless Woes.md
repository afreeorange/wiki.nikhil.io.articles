Attempted a [Netgear N300](https://wikidevi.com/wiki/Netgear_WNA3100)
with the **BCM43231** chipset, assuming that the **[bwi](http://www.freebsd.org/cgi/man.cgi?query=bwi&sektion=4)** driver could be loaded via a [tunable](http://doc.freenas.org/index.php/Tunables).

Nope. This is all I saw in `messages`

    May  4 19:03:18 Archivum kernel: ugen3.2: <Broadcom> at usbus3 (disconnected)  
    May  4 19:03:45 Archivum root: Unknown USB device: vendor 0x0846 product 0x9020 bus uhub3  
    May  4 19:03:45 Archivum kernel: ugen3.2: <Broadcom> at usbus3

Will have to research and try another card/dongle.

References
----------

-   <http://www5.us.freebsd.org/doc/handbook/config-network-setup.html>
