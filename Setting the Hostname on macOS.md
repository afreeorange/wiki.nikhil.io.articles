For my work laptop on macOS 12.6 Monterey. Assume the hostname is `dobby.ether`.

A simple `sudo hostname dobby` did not persist the hostname across sessions or reboots; it kept resetting to `MacBook-Pro`. I don't know if this was some policy pushed out or because of weirdness with macOS itself.

### HostConfig

```bash
# This did not exist
sudo vim /etc/hostconfig

# I added this line
HOSTNAME=dobby.ether
```

This _did not_ work. YMMV.

### scutil

```bash
scutil --set LocalHostName rheya
scutil --set HostName rheya.ether
```

Enter the password when asked. This _should_ work.
