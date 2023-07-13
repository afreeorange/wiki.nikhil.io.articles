Migrated from FreeBSD v11 to Ubuntu Server 22.04. These are some notes from my migration.

---

## Hardware

Read the [Community Hardware Guide](https://forums.freenas.org/index.php?resources/hardware-recommendations-guide.12/) to pick my components this time. Cheap shit causes headaches and I expect this build to last me a while.

* SuperMicro X11SSM-F-O Micro-ATX: [website](https://www.supermicro.com/products/motherboard/Xeon/C236_C232/X11SSM-F.cfm), [manual](https://public.nikhil.io/X11SSM-FO.pdf)
* Intel [Xeon E3-1230 V6 Kaby Lake](https://www.newegg.com/Product/Product.aspx?Item=N82E16819117788)
* Seasonic [FOCUS Plus Series SSR-850PX 850W 80+ Platinum](https://www.newegg.com/Product/Product.aspx?Item=N82E16817151190)
* 4 x Crucial [16GB ECC Unbuffered](http://www.crucial.com/usa/en/x11ssm-f/CT7982341) memory (`CT7982341`: DDR4-2133MHz PC4-17000 ECC Unbuffered CL15 288-Pin DIMM 1.2V Dual Rank Memory Module) - Finding additional memory was a _nightmare_.
* 5 x Western Digital [4TB Red NAS Drives](https://www.wdc.com/products/internal-storage/wd-red.html)
* [NZXT Source 210](http://www.amazon.com/gp/product/B005869A7K)
* [Rosewill RFT-120](http://www.newegg.com/Product/Product.aspx?Item=N82E16811988015) fan filters. Used nylon 8-32 × ½ Phillips flat-head screws to install them.
* An old Intel [80GB SSD Drive](https://www.amazon.com/Intel-SSDSA2CW080G3-Internal-Solid-State/dp/B00666SGRG)

Ran [`smartctl`](https://forums.freenas.org/index.php?threads/hard-drive-burn-in-testing-discussion-thread.21451/) short, conveyance, and long test with an ArchLinux ISO. [Two](https://www.orderfactory.com/articles/New-HDD-Testing.html) [more](https://github.com/Spearfoot/disk-burnin-and-testing/blob/master/disk-burnin.sh) resources on drive testing. Ran about 10 rounds of `memtest86` on the memory.

---

## Backups

Made sure to do this to two external, encrypted drives. All datasets and snapshots.

## Image

Got it [from here](https://ubuntu.com/download/server). Verified via:

```shell
echo "10f19c5b2b8d6db711582e0e27f5116296c34fe4b313ba45f9b201a5007056cb *ubuntu-22.04.1-live-server-amd64.iso" | shasum -a 256 --check
```

Formatted FAT with GUID Partition Table in _Disk Utility_ on macOS. Then used [BalenaEtcher](https://www.balena.io/etcher/) (`brew install --cask balenaetcher`) to transfer the image.

## Post Install

### Basic Stuff

```bash
sudo apt update
sudo apt upgrade

sudo apt install \
    zfsutils-linux \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-compose-plugin \
    yadm \
    ntp \
    awscli \
    neovim \
    silversearcher-ag \
    ncdu \
    fzf \
    fd-find \
    tree

# This is annoying
sudo apt remove command-not-found

# Install SSHD and change the default port
sudo apt install openssh-server

# Will install the NTP service and enable it as well.
apt install ntp
```

### NeoVim

Needed to build this from source since v0.8+ has Lua support and is pretty freaking awesome and the version of Ubuntu I installed only ships with v0.6 :/

```bash
# Install prereqs
sudo apt-get install \
    ninja-build gettext \
    libtool-bin \
    cmake \
    g++ \
    pkg-config \
    unzip \
    curl

# Build!
cd
git clone https://github.com/neovim/neovim.git
cd neovim
git checkout release-0.8
make CMAKE_BUILD_TYPE=Release
sudo make install
```

### Firewall

[Used `ufw`](/ufw_Notes) that shipped with Ubuntu. It's a wrapper around `iptables`, is simple enough, and does the trick.

### Login Logo

```

  ____  _____            _   _  _____ ______  _____ ______ _______      ________ _____
 / __ \|  __ \     /\   | \ | |/ ____|  ____|/ ____|  ____|  __ \ \    / /  ____|  __ \
| |  | | |__) |   /  \  |  \| | |  __| |__  | (___ | |__  | |__) \ \  / /| |__  | |__) |
| |  | |  _  /   / /\ \ | . ` | | |_ |  __|  \___ \|  __| |  _  / \ \/ / |  __| |  _  /
| |__| | | \ \  / ____ \| |\  | |__| | |____ ____) | |____| | \ \  \  /  | |____| | \ \
 \____/|_|  \_\/_/    \_\_| \_|\_____|______|_____/|______|_|  \_\  \/   |______|_|  \_\


```

Updated this in `/etc/update-motd.d`

### Set hostname

Screwed this up during installation.

```shell
hostnamectl set-hostname newNameHere

# Now edit this
vim /etc/hosts
```

### Netplan

System would hang on this for a while:

> [**    ] A start job is running for Wait for Network to be Configured (XX / no limit)

This is because of something called `netplan`. Saw the config in `/etc/netplan/00-installer-config.yaml`, noted that I was not using `eno2` (using `ip a`) and added `optional: true` as shown.

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    eno1:
      dhcp4: true
    eno2:
      dhcp4: true
      optional: true
  version: 2
```

This fixed the issue. I don't know what `netplan` is. I see YAML and smell 'Enterprise™'...

### Importing the Zpools

```bash
# Look for zpools
sudo zfs import

# Import a zpool named 'tank'
sudo zfs import tank -f

# To see where things are mounted. In Ubuntu's case, it will be at /tank
sudo zfs get mountpoint

# See the list of upgrades to the zpools
sudo zpool upgrade

# Perform the upgrades to ALL zpools (YOU NEED TO KNOW WHAT THIS MEANS AND WHAT
# YOU'RE DOING HERE.)
sudo zpool upgrade -a

# See when a snapshot is created. You can get other properties this way as well.
sudo zfs get creation -t snapshot tank/dataset
```

### Removing Encryption on Backup Drive

TODO: Finish this section.

```shell
# Get the ID of the drives. It will look like
#
#      gptid/66e2c3c2-2786-11ea-bc58-ac1f6b83246e.eli
#
zpool status

# Assume that my pool is called `backup`
zpool offline backup gptid/66e2c3c2-2786-11ea-bc58-ac1f6b83246e.eli
```

### Groups

For the tank

```bash
sudo groupadd wheel
sudo usermod -aG wheel nikhil
sudo usermod -aG wheel root
sudo chown -R :wheel /tank
```

ACLs were trouble, however. I had to do this to prevent "Operation not permitted" errors when I was trying to setgid with `chmod`. [This post](https://old.reddit.com/r/Proxmox/comments/socxbb/permission_denied_when_i_chmod_on_a_zfs_dataset/) helped me.

```bash
zfs set aclmode=passthrough tank/dataset

# Then this was OK
chmod g+s /tank/dataset
```

### Cron Jobs

Prefixed all of them with `custom__` to distinguish them from things that came with the system or were installed via packages.

Used this favorite as a preamble to any new jobs

```shell
# +---------------- minute (0 - 59)
# |  +------------- hour (0 - 23)
# |  |  +---------- day of month (1 - 31)
# |  |  |  +------- month (1 - 12)
# |  |  |  |  +---- day of week (0 - 6) (Sunday=0)
# |  |  |  |  |
```

### Docker

```bash
# Get the keys
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up the Docker repo
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update apt
sudo apt update
```

Got all my images via the amazing [LinuxServer.io](https://github.com/linuxserver) project.

Note that there _used_ to be a `docker-compose` command (a standalone Python script) but they made it a plugin to the `docker` command. Woo. So you omit the hyphen and rock and roll.

You will need a UID and GID that the containers run as. On a base Ubuntu Server system, these are `1000` and `1000` which map to the non-root user the installer had you create.

Created separate configurations for each of the services.

```bash
# Start the container with the service name in the background (-d)
docker compose -f /path/to/service.yml -p service_name up -d

# Stop the container
docker compose -f /path/to/service.yml down

# Restart the container
docker compose -f /path/to/service.yml restart

# Tail logs
docker logs --follow <Container ID>

# Check if a container is set to start at boot
docker inspect <container_id> | restart -A 5
```

When you make changes to the configuration YAML files, merely restarting the containers will not apply the changes! Say you made a change to the Plex server

```bash
# Find out the name of the service
docker ps

# Assume it's called `plex`. Now stop it
docker compose -f /orangepool/docker/plex.yml -p plex stop

# Make your changes to the config YAML, and then bring **another** container back up!
docker compose -f /orangepool/docker/plex.yml -p plex up -d

# **** THIS WILL NOT APPLY CHANGES! ****
docker compose -f /orangepool/docker/plex.yml -p plex start
````

[Review this](https://stackoverflow.com/a/67335047) for the differences between `always`, `unless-stopped`, etc.

Networking was a bit iffy. I want the containers to appear as if they were on the LAN. `macvlan` networks appeared to be the answer. There are two modes: Bridge and 802.1q Trunk Bridge. I tried the former.

```bash
# Create a simple bridge. The 802.1q stuff is a complicated PITA for your use-case...
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eno1 \
  my-macvlan

# Make sure it exists
docker network ls

# Inspect it
docker network inspect <Network ID>
```

And now you add the bridge on the _host_ so it knows how to route traffic.

```bash
# ip link add $INTERFACE_NAME link $PARENTDEV type macvlan mode bridge
ip link add mac0 link eno1 type macvlan mode bridge

# Assign an IP to this interface
ip addr add 192.168.1.100/32 dev mac0

# Bring up the interface
ip link set mac0 up

# Tell docker to use the interface to communicate
ip route add 192.168.1.0/24 dev mac0
```

These should persist across reboots. People [configure stuff](https://old.reddit.com/r/docker/comments/amle59/docker_macvlan_setup_ubuntu_no_ip_address_on/) in `/etc/network/interface` but I didn't need to do that (which mystifies me...)

Some containers didn't have the `ip` command. Install via `apt install iproute2`.

Constrained on router to assign same IP based on MAC. Docker appears to assign `02:42:0a` as the OUI. So be it.

[This is how you specify an external default network](https://github.com/docker/compose-cli/issues/1856#issuecomment-1108552064) in Docker Compose.

### Desktop Environment

Meant for this to be headless but I am a lazy person. Used my lovely XFCE4 and [TigerVNC](https://tigervnc.org/). Two important notes:

1. You'll have to restart the VNC Server if you log out of your session. There [are solutions](https://unix.stackexchange.com/questions/43398/is-it-possible-to-keep-a-vnc-server-alive-after-log-out) but I did not use them.
2. This configuration will only allow connections on `localhost`. You'll need an SSH tunnel to the server.

```bash
sudo apt install xfce4 tigervnc-standalone-server firefox

# Set the password for the current (NON-ROOT) user
vncpasswd

# Upon connection via VNC, start XFCE4
cat <<EOF > ~/.vnc/xstartup
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4

EOF
chmod u+x ~/.vnc/xstartup

# Create your own preferences for startup
cat <<EOF > ~/.vnc/config
geometry=1280x960
depth=32

EOF

# Now create a service for the VNC Server. As root:
cat <<EOF > /etc/systemd/system/vncserver@.service
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

# DO NOT BE ROOT FFS. The '-localhost' flag is important too.
[Service]
Type=simple
User=nikhil
Group=wheel
WorkingDirectory=/home/nikhil

ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill :%i > /dev/null 2>&1 || :'
ExecStart=/usr/bin/vncserver -localhost :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target

EOF

# Tell systemd about it and bring it up at reboot. This is a single instance btw.
sudo systemctl daemon-reload
sudo systemctl enable vncserver@1
sudo systemctl start vncserver@1

# See some startup logs
journalctl -u vncserver@1 -f

# Now on the client, create an SSH tunnel
ssh -p 3227 -L 5901:127.0.0.1:5901 -N -f -l nikhil 192.168.1.10
```

Note that you will have to adjust the bit depth/quality on your viewer. This is not something TigerVNC controls.

### Samba

[This](https://superuser.com/a/1400920) was an excellent guide! Also see [this](https://linuxg.net/how-to-set-the-setuid-and-setgid-bit-for-files-in-linux-and-unix/).

Setting up Samba shares was rather easy in `/etc/samba/smb.conf`. Make sure you [do this to prevent Guest Access](https://unix.stackexchange.com/a/562999).

### Sanoid

[Nice little Perl script](https://github.com/jimsalterjrs/sanoid) to automate ZFS snapshotting. `apt install sanoid` installs a `cron` script in `/etc/cron.d/sanoid`. Configuation in `/etc/sanoid/sanoid.conf` was as simple as this. I want snapshots of all child datasets _except_ for one and this is what my config looked like ([reference](https://github.com/jimsalterjrs/sanoid/issues/627#issue-837657066)).

```
[tank]
  use_template = base
  recursive = yes

[tank/dataset-to-exclude]
  use_template = none

# --- Templates ---

[template_base]
  autoprune = yes
  autosnap = yes
  daily = 30
  frequently = 0
  hourly = 0
  monthly = 12
  yearly = 1

[template_none]
  autoprune = no
  autosnap = no
  daily = 0
  frequently = 0
  hourly = 0
  monthly = 0
  yearly = 0
```

See [the options](https://github.com/jimsalterjrs/sanoid/wiki/Sanoid#options) for more info on each. Note that the defaults are `/usr/share/sanoid/sanoid.defaults.conf`.

### Glances

Type `glances -V` to see some info about the version and the log file. The one installed via `apt` needed some surgery.

```bash
export GLANCES_VERSION="3.2.4.2"
wget https://github.com/nicolargo/glances/archive/refs/tags/v${GLANCES_VERSION}.tar.gz
tar zxvf v${GLANCES_VERSION}.tar.gz
sudo cp -r glances-${GLANCES_VERSION}/glances/outputs/static/public/ /usr/lib/python3/dist-packages/glances/outputs/static
```

Here's the service definition

```bash
# /etc/systemd/system/glances.service
[Unit]
Description = Glances in Web Server Mode
After = network.target

[Service]
ExecStart = /usr/bin/glances -w -t 5 -B 0.0.0.0 -p 8080

[Install]
WantedBy = multi-user.target
```

## Miscellaneous Notes

- I use DuckDNS for Dynamic DNS
- No password on PiHole via `sudo pihole -a -p` followed by an <kbd>Enter</kbd>. It's how you set a password as well.
- Found myself not being able to run more than two containers at once. [Docker Compose is strange](https://stackoverflow.com/a/39489432). Each `docker compose up` command needs to be given its own `-p`
- To list the contents of a package, `dpkg-query -L package-name`
- Needed to add this to get `fzf` working

```bash
# shellcheck source=/dev/null
if [[ $(uname) == "Linux" ]]; then
  [[ -f /usr/share/doc/fzf/examples/key-bindings.bash ]] && source /usr/share/doc/fzf/examples/key-bindings.bash
fi
```

### Reset HomeBridge password

Remove `auth.json` in `/var/lib/homebridge` if installed on a RaspberryPi.
Else you can find it in `$HOME/.homebridge`.
Else you can find it in `/var/lib/docker/volumes/homebridge` if running in Docker.
(Else just reinstall the damn thing and keep better track of your passwords...)

Then `sudo systemctl restart homebridge.service`. It's on `http://192.168.1.75:8581`

### exFAT on Backup Drive

Wanted to use an 8TB external drive for backups. Didn't have much luck getting stuff formatted by `gparted` and `fdisk` and `mkfs` to work on macOS (i.e., it wouldn't recognize the partitions on the drive) so just used Disk Utility to format it.

```bash
# List all physical disks. The flag removes loops devices.
lsblk -e7

# or, for a longer and better view,
fdisk -l

# Mount the disk
mount -o uid=1000,gid=1001,umask=002 /dev/sdg2 /backup-02
```

`rsync` with the `-a` flag will preserve permissions. This would give me `chgrp operation not permitted` errors. This is because exFAT does not support permissions 🤷‍♂️. So,

```bash
# I'm lazy and don't want to look up the exact options. Hence the
# "add everything and remove what I don't need" flags here:

rsync -avWHh --no-perms --no-owner --no-group --progress /source/ /backup/
```

### Remove Snap

Runs in containers, checks for updates four times a day without my permission, is another thing Canonical can stop forcing us to use. Sources: [1](http://blog.pagefault-limited.co.uk/remove-snapd-from-ubuntu-kubuntu-20-04-and-restore-chromium-apt-deb-package), [2](https://haydenjames.io/remove-snap-ubuntu-22-04-lts/), [3](https://www.baeldung.com/linux/snap-remove-disable)

```bash
# Stop the service. You will have errors removing each snap if you don't.
sudo systemctl stop snapd

# List things to remove
snap list | grep -v "^Name" | awk {'print "sudo snap remove " $1'}

# Remove each one.
sudo snap remove bare
sudo snap remove core20
sudo snap remove core22
sudo snap remove cups
sudo snap remove gnome-3-38-2004
sudo snap remove gtk-common-themesÏ
sudo snap remove lxd
sudo snap remove snapd

# Disable things
sudo systemctl disable snapd.service
sudo systemctl disable snapd.socket
sudo systemctl disable snapd.seeded.service
sudo systemctl disable snapd.autoimport.service
sudo systemctl disable snapd.apparmor.service

# AppArmor things
sudo rm -rf /etc/apparmor.d/usr.lib.snapd.snap-confine.real
sudo systemctl start apparmor.service

# Find a list of snap mounts and remove them
df | grep snap | awk {'print "sudo umount " $6'}

# Purge snap
sudo apt purge snapd

# Clean things up
rm -rf ~/snap
sudo rm -rf /snap
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd

# Create configs for future installs like Firefox and Chromium
cat <<EOF | sudo tee /etc/apt/preferences.d/snapd
Package: snapd
Pin: origin *
Pin-Priority: -1
EOF

# cat <<EOF | sudo tee /etc/apt/preferences.d/pin-xalt7x-chromium-deb-vaapi
# Package: *
# Pin: release o=LP-PPA-xalt7x-chromium-deb-vaapi
# Pin-Priority: 1337
# EOF
# sudo add-apt-repository ppa:xalt7x/chromium-deb-vaapi

cat <<EOF | sudo tee /etc/apt/preferences.d/firefox-no-snap
Package: firefox*
Pin: release o=Ubuntu*
Pin-Priority: -1
EOF
sudo add-apt-repository ppa:mozillateam/ppa

# Install things
sudo apt update
sudo apt install firefox
sudo apt install chromium-browser
```

## References

- [Migrating FreeNAS ZFS Pools to Ubuntu](https://drechsel.xyz/posts/how-to-migrate-existing-freenas-zfs-pools-to-ubuntu-1804-and-higher/)
- [Fantastic Overview of macvlans](https://gdevillele.github.io/engine/userguide/networking/get-started-macvlan/)
- https://www.linuxtechi.com/create-use-macvlan-network-in-docker/
- https://github.com/sarunas-zilinskas/docker-compose-macvlan/blob/master/docker-compose.yml
- https://runnable.com/docker/docker-compose-networking
- https://collabnix.com/2-minutes-to-docker-macvlan-networking-a-beginners-guide/
- [Ansible NAS](https://github.com/davestephens/ansible-nas)
