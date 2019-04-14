### Installing Plex

Did this in a FreeBSD jail for the FreeNAS plugin sucks.

```bash
# Update system and install Plex
pkg update
pkg upgrade
pkg install plexmediaserver-plexpass

# Installed here (if you have a PlexPass)
/usr/local/share/plexmediaserver-plexpass

# Enable at boot
sysrc plexmediaserver_plexpass_enable=YES
```

### Migrating Data

Data is kept here

```bash
# FreeBSD
/usr/local/plexdata-plexpass

# Plex plugin
/var/db/plexdata/Plex Media Server
```

Sync that stuff to the new server before you start the Plex Daemon:

```bash
service plexmediaserver_plexpass start
```

Then

* Log out and log back in
* Clean Bundles, Empty Trash, Optimize Database
* Make sure the server's accessible outside the network
* Make sure Plex comes up when you reboot

References
----------

* [Where is the Plex Media Server data directory located?](https://support.plex.tv/articles/202915258-where-is-the-plex-media-server-data-directory-located/)
* [Moving your Plex install to another system](https://support.plex.tv/articles/201370363-move-an-install-to-another-system/)
