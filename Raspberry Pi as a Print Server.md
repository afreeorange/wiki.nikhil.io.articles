On a Raspberry Pi 3 running Raspbian GNU/Linux 9 (stretch). Works fine on macOS (you won't see the pretty printer icon that looks like your printer.)

```bash
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

Here's the diff

```
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

* The server is accessible from the LAN
* I added `Allow @Local` to the three `<Location>` blocks

```bash
# Restart CUPS
sudo service cups restart

# Allow the `pi` user to administer CUPS
# This means that when you see HTTP Basic auth
# when trying to add/administer printers,
# you simply use the `pi` user creds.
sudo usermod -a -G lpadmin pi
```

Navigate to `http://pi:631`. You'll know what to do. 
