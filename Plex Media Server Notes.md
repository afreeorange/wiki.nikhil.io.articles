Installation
------------

Did this in a FreeBSD jail for the FreeNAS plugin sucks.

```bash
# Get more current updates to PMS
mkdir -p /usr/local/etc/pkg/repos
touch /usr/local/etc/pkg/repos/FreeBSD.conf
```

Add this to `FreeBSD.conf`

```
FreeBSD: {
    url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```

```bash
# Update system and install Plex
pkg update -f
pkg upgrade -f
pkg install plexmediaserver-plexpass

# Installed here (if you have a PlexPass)
/usr/local/share/plexmediaserver-plexpass

# Enable at boot
sysrc plexmediaserver_plexpass_enable=YES
```

Migration
---------

Data is kept here

```bash
# FreeBSD
/usr/local/plexdata-plexpass/Plex Media Server  

# Plex plugin
/var/db/plexdata/Plex Media Server
```

Sync that stuff to the new server before you start the Plex Daemon:

```bash
service plexmediaserver_plexpass start
```

Then go to the Plex Web UI

* Log out and log back in
* Settings -> Manage -> Troubleshooting: Clean Bundles, Empty Trash, Optimize Database
* Settings -> Remote Access: Make sure the server's accessible outside the network

Downgrading
-----------

Had to downgrade from `v1.18.4.2171` to the last version that worked `v1.17.0.1709`. This was because I'd keep seeing the wonderful "_The transcoder exited due to an error_" message only on my Samsung TV. Nothing in the logs (except for the fact that the newer version required a `/home/plex/transcode` which didn't exist; creating this folder didn't help.) Gave up trying to solve the problem.

```bash
# Stop Plex
service plexmediaserver_plexpass stop

# Remove latest version
pkg remove plexmediaserver-plexpass

# Add the last one that works
pkg add /var/cache/pkg/plexmediaserver-plexpass-1.17.0.1709.txz

# Start Plex
service plexmediaserver_plexpass start
```

References
----------

* [Where is the Plex Media Server data directory located?](https://support.plex.tv/articles/202915258-where-is-the-plex-media-server-data-directory-located/)
* [Moving your Plex install to another system](https://support.plex.tv/articles/201370363-move-an-install-to-another-system/)
