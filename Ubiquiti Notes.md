### Installing the UniFi Controller in a Jail

Super easy! In a fresh jail,

```bash
pkg update
pkg install unifi5
sysrc unifi_enable=YES
service unifi start
```

Note that this will most certainly be a few versions behind [upstream](https://www.ui.com/download/unifi/).

### Clear DNS Cache

Go to Site &rarr; Services &rarr; Advanced Features and enable it. Hit "Save". You'll see a new section called "Device Authentication" at the bottom. It will have the SSH credentials. SSH into the USG.

```bash
# Inspect and remove any offending entries
sudo vi /etc/hosts

# Clear the cache
clear dns forwarding cache
```
