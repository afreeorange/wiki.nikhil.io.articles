On a Raspberry Pi 3 running Raspbian GNU/Linux 9 (stretch). Works fine on macOS (you won't see the pretty printer icon that looks like your printer.)

```bash
# Check if the printer is connected
lsusb

# Update stuff
sudo apt-get update
sudo apt-get upgrade

# Install CUPS and reboot
sudo apt-get install cups
sudo reboot

# Create a backup of the CUPS config
sudo cp /etc/cups/cupsd.conf{,.default}

# Start editing
sudo vi /etc/cups/cupsd.conf
```

Here's the diff. You'll just say `Listen 631` so you'll listen on all interfaces (i.e. you can access this on the LAN). I added `Allow @Local` to all `<Location>` blocks _towards the end_.

```diff
16,17c16
< #Listen localhost:631
< Listen 631
---
> Listen localhost:631
33d31
<   Allow @Local
39d36
<   Allow @Local
54d50
<   Allow @Local
```

Now,

```bash
# Restart CUPS
sudo service cups restart

# Allow the `pi` user to administer CUPS
# This means that when you see HTTP Basic auth
# when trying to add/administer printers,
# you simply use the `pi` user creds (or
# whatever your username is).
sudo usermod -a -G lpadmin pi
```

Navigate to `http://pi:631`. Modify the printer to share it on the network.

### AirPrint Support

Easier than I thought it would be!

```bash
sudo apt-get install avahi-discover
systemctl is-enabled avahi-daemon.service
```

When you change the printer's name, you must restart this service.

### Firewall

Update `ufw` if needed

```bash
# Update firewall for CUPS and Avahi
sudo ufw allow 631/tcp
sudo ufw allow 5353/udp
```

### Miscellaneous

You'll need `system-config-printer` installed if you want to access printer settings via a GUI.

Don't do this. Do it as a last resort I guess:

```bash
# For a home server inside a DMZ, this is the simplest. Will not require
# auth. Else you have to add a dedicated user to the lpadmin group AND
# set a password!
cupsctl --remote-admin --remote-any
```
