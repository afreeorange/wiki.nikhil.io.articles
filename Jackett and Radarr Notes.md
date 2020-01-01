Attempted install in a FreeNAS jail (not using the Radarr plugin.)

```bash
pkg update

# These will be behind the latest versions
pkg install radarr jacket

# Startup
echo "radarr_enable=\"YES\"" >> /etc/rc.conf
echo "jackett_enable=\"YES\"" >> /etc/rc.conf

# Radarr: Avoid permissions issues when upgrading
ln -s /usr/local/bin/mono /bin
chown -R radarr:radarr /usr/local/share/radarr

# Jackett: Upgrade to the latest package. Check GitHub
# for the latest releases
#
# https://github.com/Jackett/Jackett/releases
fetch https://github.com/Jackett/Jackett/releases/download/v0.12.1391/Jackett.Binaries.Mono.tar.gz -o /usr/local/share
tar -xzvf /usr/local/share/Jackett.Binaries.Mono.tar.gz -C /usr/local/share
mv /usr/local/share/Jackett /usr/local/share/jackett
mv /usr/local/share/jackett /usr/local/share/jackett-old
rm /usr/local/share/Jackett.Binaries.Mono.tar.gz

# Restart jail
```
