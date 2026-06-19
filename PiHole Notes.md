## Maintenance

```bash
# Upgrade PiHole and Gravity lists
pihole -up && pihole -g

# Restart DNS Subsystem
pihole restartdns

# If it complains about an old version of the OS, skip the check
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true sudo -E bash
```

## Logs

```bash
tail -f /var/log/pihole.log
```

## Re/Set Password

```bash
# If you hit enter here, the web interface will not require a password
pihole setpassword
```

## New Lists

To install, go to Settings -> Blocklists and add them there. Then go to Tools -> Update Gravity.

Historical Note: These _used_ to go in `/etc/pihole/adlists.list` but, since PiHole 5+ will use SQLite instead of flat text files, you gotta add these via the UI.

### Adding lists via the CLI

Could not find a way to do this...

### Regexes

Here's [an overview](https://docs.pi-hole.net/ftldns/regex/overview/). Added [this list](https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list).

### Additional Blocklists

Find a [big list here](https://firebog.net/). Find the "ticked" lists [here](https://v.firebog.net/hosts/lists.php) (these are safe to add and won't cause issues.)

### SmartTV Blocklists

See [this GitHub gist](https://github.com/Perflyst/PiHoleBlocklist/blob/master/SmartTV.txt). I have Samsung TVs, so some caveats apply about blocking domains like `cdn.samsungcloudsolution.com` and (especially) `time.samsungcloudsolution.com`.

### Google AMP 🙄

Add [this list](https://www.github.developerdan.com/hosts/lists/amp-hosts-extended.txt).

## Using PiHole Locally on macOS

This is for M-Series macs. Use [UTM](https://mac.getutm.app/) to boot up [the `aarch64` ISO of Alpine Linux](https://www.alpinelinux.org/downloads/) on a VM with 256GB memory and 2GB disk space and _shared networking_ (I used `alpine-standard-3.24.1-aarch64.iso`.) Setting all this up weighed in at around 550MB.

**NOTE**: Setting this to lower memory leads to weirdness. You either won't be able to get past the UEFI boot menu or will see this:

```
error:
/home/buildozer/aports/main/grub/src/grub-2.14/grub-core/loader/efi/linux.c:gru b_arch_efi_linux_boot_image:259:start_image0 returned 0x8000000000000009.
Press any key to continue...
```

Type `root` and hit enter twice (there's no password.) Run `setup-alpine`. Use all defaults. Create a new user `pihole:pihole` (user:password). Then remove the ISO and reboot. Log in as `root` and run these:

```bash
apk update
apk upgrade
apk add bash curl wget git procps newt python3 sudo openssl
curl -sSL https://install.pi-hole.net | bash

# I set the password to none.
pihole setpassword
```

You'll need a static IP. This is what `/etc/network/interfaces` looks like:

```
auto lo
iface lo inet loopback
iface lo inet6 loopback

auto eth0
iface eth0 inet static
	address 192.168.64.4
	mask 255.255.255.0
	gateway 192.168.64.1
iface eth0 inet6 auto
```

Restart networking with `rc-service networking restart`.

### Headless UTM

[Official Docs](https://docs.getutm.app/advanced/headless/). You need to right-click the VM -> Edit and then right-click and remove the Display (and Sound while you're at it.) Then,

```bash
sudo ln -sf /Applications/UTM.app/Contents/MacOS/utmctl /usr/local/bin/utmctl

# Then see what's running
utmctl list

# Start shit up
utmctl start pihole
```

