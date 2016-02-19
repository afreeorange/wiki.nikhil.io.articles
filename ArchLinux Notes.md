[TOC]

Notes from installing the awesome ArchLinux on VirtualBox to use as a development machine at work.

```bash
parted /dev/sda

# Make a GPT partition table
mklabel msdos

# Create 2GiB swap
mkpart primary linux-swap 1MiB 2GiB

# Use the rest for root
mkpart primary ext4 2GiB 100%

# Make root bootable
set 2 boot on

# Ctrl+D to quit

# Create swap
mkswap /dev/sda1
swapon /dev/sda1

# Create filesystem
mkfs.ext4 /dev/sda2

# Mount
mount /dev/sda2 /mnt

# Boostrap
pacstrap -i /mnt base base-devel

# Generate fstab
genfstab -U /mnt > /mnt/etc/fstab

# Switch to the new filesystem!
arch-chroot /mnt /bin/bash
```

Then do everything else [the wiki asks you to do](https://wiki.archlinux.org/index.php/beginners'_guide#Locale)

Install VirtualBox Guest Additions
----------------------------------

    pacman -S virtualbox-guest-utils \
              virtualbox-guest-modules \
              virtualbox-guest-dkms

This is without an LTS kernel since I couldn't be bothered. After installation, enable the service

    systemctl enable vboxservice.service

Edit `/etc/modules-load.d/virtualbox.conf` to add these

    vboxguest 
    vboxsf
    vboxvideo

Install X11 and Xfce4
---------------------

    pacman -S xorg-server xorg-xinit xfce4 xfce4-goodies

At this point, running `startxfce4` should show you a desktop. Reboot.

Starting X
----------

    cp /etc/X11/xinit/xinitrc ~/.xinitrc
    echo -e "exec startxfce4" >> ~/.xinitrc

Modify `~/.xinitrc` to remove all the `xterm`, `xclock` and `exec` lines and add this

    exec startxfce4

Now, `startx` should work!

Install some extras

    yaourt -S google-chrome numix-themes numix-circle-icon-theme-git ristretto evince2-light squeeze-git --noconfirm

Add a normal user
-----------------

Who will be able to `sudo` and do things and set a password

    useradd -m -g wheel user
    passwd user

Can always change the name of this user later with `usermod`. Then use `visudo` to enable the `wheel` group. You'll see a lot of artifacts if you don't set `EDITOR` first

    EDITOR=vim visudo

Yoghurt
-------

Edit `/etc/pacman.conf` and add this

    [archlinuxfr]
    SigLevel = Never
    Server = http://repo.archlinux.fr/$arch

Then,

    pacman -Sy yaourt

Compiz (Maybe)
--------------

For Compiz,

    yaourt -S compiz

To run compiz,

    compiz --replace ccp

To get the [Numix theme](https://wiki.archlinux.org/index.php/Compiz_configuration#Window_decoration_themes),

    gsettings set org.gnome.metacity theme theme-name

Add that to "Session and Startup". I had to kill it, _not save the session_, and log out. The default window manager is `xfwm4`.

The Trash Can
-------------

    sudo pacman -S gvfs gamin

Sound
-----

    pacman -S alsa-firmware alsa-utils
    alsactl init

Dock
----

Lots of options, but I like Docky and Plank. Went with Plank. 

Dropbox
-------
 
Install both the `dropbox` and `dropbox-cli` packages with `yaourt`. Some useful commands

```bash
# Get to your folder
cd ~/Dropbox

# See the overall sync status
dropbox-cli status

# See file status
dropbox-cli filestatus

# Set a proxy
dropbox-cli proxy manual http jhproxy1.phibred.com 8080
```

NetworkManager
--------------

A bit 'heavy' compared to `netctl` but I was tired of fighting with the corporate network.

    pacman -S networkmanager network-manager-applet xfce4-notifyd

Enable the service (else you'll get D-Bus errors when you run `nm-applet`)

    systemctl enable NetworkManager.service    
    systemctl start NetworkManager.service    

Reboot and log back in. You'll find the network manager in Applications -> Settings -> Network Connections

Verbose Boot
------------

Modify `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`

CA Certificates
---------------

Copy certificates in PEM format _and ending with a `.pem` extension_ to `/etc/ssl/certs`. Then, as root, run `update-ca-trust`.

Google Chrome didn't seem to depend on the system store.

Windows Fonts
-------------

From a Windows 7 system. Get them and rename so you can remove later if you'd like (`for f in *; do mv $f "Win7-"$f; done`)

```bash
@ECHO OFF
CLS
SET MYSHARE=%USERPROFILE%\Desktop\TTFONTS-ttf-win7-fonts
MKDIR "%MYSHARE%"
ECHO 1
COPY "%WINDIR%\FONTS\arial.ttf" "%MYSHARE%\arial.ttf"
COPY "%WINDIR%\FONTS\arialbd.ttf" "%MYSHARE%\arialbd.ttf"
COPY "%WINDIR%\FONTS\ariali.ttf" "%MYSHARE%\ariali.ttf"
COPY "%WINDIR%\FONTS\arialbi.ttf" "%MYSHARE%\arialbi.ttf"
COPY "%WINDIR%\FONTS\comic.ttf" "%MYSHARE%\comic.ttf"
COPY "%WINDIR%\FONTS\comicbd.ttf" "%MYSHARE%\comicbd.ttf"
COPY "%WINDIR%\FONTS\cour.ttf" "%MYSHARE%\cour.ttf"
COPY "%WINDIR%\FONTS\courbd.ttf" "%MYSHARE%\courbd.ttf"
COPY "%WINDIR%\FONTS\couri.ttf" "%MYSHARE%\couri.ttf"
COPY "%WINDIR%\FONTS\courbi.ttf" "%MYSHARE%\courbi.ttf"
COPY "%WINDIR%\FONTS\gabriola.ttf" "%MYSHARE%\gabriola.ttf"
COPY "%WINDIR%\FONTS\georgia.ttf" "%MYSHARE%\georgia.ttf"
COPY "%WINDIR%\FONTS\georgiab.ttf" "%MYSHARE%\georgiab.ttf"
COPY "%WINDIR%\FONTS\georgiai.ttf" "%MYSHARE%\georgiai.ttf"
COPY "%WINDIR%\FONTS\georgiaz.ttf" "%MYSHARE%\georgiaz.ttf"
COPY "%WINDIR%\FONTS\impact.ttf" "%MYSHARE%\impact.ttf"
COPY "%WINDIR%\FONTS\times.ttf" "%MYSHARE%\times.ttf"
COPY "%WINDIR%\FONTS\timesbd.ttf" "%MYSHARE%\timesbd.ttf"
COPY "%WINDIR%\FONTS\timesi.ttf" "%MYSHARE%\timesi.ttf"
COPY "%WINDIR%\FONTS\timesbi.ttf" "%MYSHARE%\timesbi.ttf"
COPY "%WINDIR%\FONTS\trebuc.ttf" "%MYSHARE%\trebuc.ttf"
COPY "%WINDIR%\FONTS\trebucbd.ttf" "%MYSHARE%\trebucbd.ttf"
COPY "%WINDIR%\FONTS\trebucit.ttf" "%MYSHARE%\trebucit.ttf"
COPY "%WINDIR%\FONTS\trebucbi.ttf" "%MYSHARE%\trebucbi.ttf"
ECHO 25
COPY "%WINDIR%\FONTS\verdana.ttf" "%MYSHARE%\verdana.ttf"
COPY "%WINDIR%\FONTS\verdanab.ttf" "%MYSHARE%\verdanab.ttf"
COPY "%WINDIR%\FONTS\verdanai.ttf" "%MYSHARE%\verdanai.ttf"
COPY "%WINDIR%\FONTS\verdanaz.ttf" "%MYSHARE%\verdanaz.ttf"
COPY "%WINDIR%\FONTS\webdings.ttf" "%MYSHARE%\webdings.ttf"
COPY "%WINDIR%\FONTS\wingding.ttf" "%MYSHARE%\wingding.ttf"
COPY "%WINDIR%\FONTS\sylfaen.ttf" "%MYSHARE%\sylfaen.ttf"
COPY "%WINDIR%\FONTS\symbol.ttf" "%MYSHARE%\symbol.ttf"
COPY "%WINDIR%\FONTS\calibri.ttf" "%MYSHARE%\calibri.ttf"
COPY "%WINDIR%\FONTS\calibril.ttf" "%MYSHARE%\calibril.ttf"
COPY "%WINDIR%\FONTS\calibrib.ttf" "%MYSHARE%\calibrib.ttf"
COPY "%WINDIR%\FONTS\calibrii.ttf" "%MYSHARE%\calibrii.ttf"
COPY "%WINDIR%\FONTS\calibrili.ttf" "%MYSHARE%\calibrili.ttf"
COPY "%WINDIR%\FONTS\calibriz.ttf" "%MYSHARE%\calibriz.ttf"
COPY "%WINDIR%\FONTS\cambria.ttc" "%MYSHARE%\cambria.ttc"
COPY "%WINDIR%\FONTS\cambriab.ttf" "%MYSHARE%\cambriab.ttf"
COPY "%WINDIR%\FONTS\cambriai.ttf" "%MYSHARE%\cambriai.ttf"
COPY "%WINDIR%\FONTS\cambriaz.ttf" "%MYSHARE%\cambriaz.ttf"
COPY "%WINDIR%\FONTS\candara.ttf" "%MYSHARE%\candara.ttf"
COPY "%WINDIR%\FONTS\candarab.ttf" "%MYSHARE%\candarab.ttf"
COPY "%WINDIR%\FONTS\candarai.ttf" "%MYSHARE%\candarai.ttf"
COPY "%WINDIR%\FONTS\candaraz.ttf" "%MYSHARE%\candaraz.ttf"
COPY "%WINDIR%\FONTS\consola.ttf" "%MYSHARE%\consola.ttf"
COPY "%WINDIR%\FONTS\consolab.ttf" "%MYSHARE%\consolab.ttf"
ECHO 49
COPY "%WINDIR%\FONTS\consolai.ttf" "%MYSHARE%\consolai.ttf"
COPY "%WINDIR%\FONTS\consolaz.ttf" "%MYSHARE%\consolaz.ttf"
COPY "%WINDIR%\FONTS\constan.ttf" "%MYSHARE%\constan.ttf"
COPY "%WINDIR%\FONTS\constanb.ttf" "%MYSHARE%\constanb.ttf"
COPY "%WINDIR%\FONTS\constani.ttf" "%MYSHARE%\constani.ttf"
COPY "%WINDIR%\FONTS\constanz.ttf" "%MYSHARE%\constanz.ttf"
COPY "%WINDIR%\FONTS\corbel.ttf" "%MYSHARE%\corbel.ttf"
COPY "%WINDIR%\FONTS\corbelb.ttf" "%MYSHARE%\corbelb.ttf"
COPY "%WINDIR%\FONTS\corbeli.ttf" "%MYSHARE%\corbeli.ttf"
COPY "%WINDIR%\FONTS\corbelz.ttf" "%MYSHARE%\corbelz.ttf"
COPY "%WINDIR%\FONTS\lucon.ttf" "%MYSHARE%\lucon.ttf"
COPY "%WINDIR%\FONTS\ariblk.ttf" "%MYSHARE%\ariblk.ttf"
COPY "%WINDIR%\FONTS\l_10646.ttf" "%MYSHARE%\l_10646.ttf"
COPY "%WINDIR%\FONTS\micross.ttf" "%MYSHARE%\micross.ttf"
COPY "%WINDIR%\FONTS\pala.ttf" "%MYSHARE%\pala.ttf"
COPY "%WINDIR%\FONTS\palab.ttf" "%MYSHARE%\palab.ttf"
COPY "%WINDIR%\FONTS\palai.ttf" "%MYSHARE%\palai.ttf"
COPY "%WINDIR%\FONTS\palabi.ttf" "%MYSHARE%\palabi.ttf"
COPY "%WINDIR%\FONTS\tahoma.ttf" "%MYSHARE%\tahoma.ttf"
COPY "%WINDIR%\FONTS\tahomabd.ttf" "%MYSHARE%\tahomabd.ttf"
COPY "%WINDIR%\FONTS\framd.ttf" "%MYSHARE%\framd.ttf"
COPY "%WINDIR%\FONTS\framdit.ttf" "%MYSHARE%\framdit.ttf"
COPY "%WINDIR%\FONTS\segoepr.ttf" "%MYSHARE%\segoepr.ttf"
COPY "%WINDIR%\FONTS\segoeprb.ttf" "%MYSHARE%\segoeprb.ttf"
ECHO 73
COPY "%WINDIR%\FONTS\segoesc.ttf" "%MYSHARE%\segoesc.ttf"
COPY "%WINDIR%\FONTS\segoescb.ttf" "%MYSHARE%\segoescb.ttf"
COPY "%WINDIR%\FONTS\segoeui.ttf" "%MYSHARE%\segoeui.ttf"
COPY "%WINDIR%\FONTS\segoeuib.ttf" "%MYSHARE%\segoeuib.ttf"
COPY "%WINDIR%\FONTS\segoeuii.ttf" "%MYSHARE%\segoeuii.ttf"
COPY "%WINDIR%\FONTS\segoeuil.ttf" "%MYSHARE%\segoeuil.ttf"
COPY "%WINDIR%\FONTS\segoeuiz.ttf" "%MYSHARE%\segoeuiz.ttf"
COPY "%WINDIR%\FONTS\seguisb.ttf" "%MYSHARE%\seguisb.ttf"
COPY "%WINDIR%\FONTS\seguisym.ttf" "%MYSHARE%\seguisym.ttf"
PAUSE
```

---

[Linode](http://linode.com) Notes
---------------------------------

## Separating out partitions

I had separate disks for `/var` and `/var/log`. 

* Create a minimal system
* Boot into *Rescue Mode* (that uses [Finnix](http://www.finnix.org/))
* Mount the minimal system some place
* Modify `/etc/fstab` on the system

        /dev/sda            /           ext4        rw,relatime,data=ordered    0 1
        /dev/sdb            none        swap        defaults    0 0
        /dev/sdc            /var        ext4        rw,relatime,data=ordered    0 1
        /dev/sdd            /var/log    ext4        rw,relatime,data=ordered    0 1
        tmpfs               /tmp        tmpfs       nodev,nosuid                0 0

* `rsync` everything over from the minimal system's folders to the new disk
* Delete folders from the minimal system
* Shutdown the minimal system (`shutdown now`)
* Make sure that the mounts are correctly mapped in your *Configuration Profile*
* Boot up the minimal system

## Packages

*   Run `pacman -Syu` first!
*   The `base-devel` collection isn't installed. A simple `pacman -S base-
    devel` will fix this.

## Network

The `pacman` update will break networking due [a 
bug](https://bugs.archlinux.org/task/41215) that may have been fixed in 
`systemd` v228 (as of this writing). Oh well. The fix is easy. Create a file 
[like this](https://wiki.archlinux.org/index.php/Systemd-networkd#Wired_adapter_using_DHCP) 
for the interface you see in `ip link` (will start with "`en`")

    # /etc/systemd/network/enp0s4.network
    [Match]
    Name=enp0s4

    [Network]
    DHCP=yes

Then enable the appropriate service and restart the node

    systemctl enable systemd-networkd
    reboot

## Hostname

    hostnamectl set-hostname nikhil.io

### SSH

    pacman -S openssh

Change default port in `/etc/ssh/sshd_config` and disable root login. Then 
[enable the "spawn on demand" `ssh.socket` service](https://wiki.archlinux.org/index.php/Secure_Shell#Daemon_management)
and change the port to whatever you had earlier

    # systemctl edit sshd.socket
    [Socket]
    ListenStream=12345

Enable the service and reboot to test if you can SSH

    systemctl enable sshd.socket
    reboot

Time and Date
-------------

    timedatectl set-timezone America/Chicago

Installation
------------

[Downloaded](https://www.archlinux.org/download/) the ISO (`2015.12.01`) and
set up an "Other 64-bit" VM in VMWware Fusion 8 on OS X El Capitan. Wired
networking was working at bootup. The Arch [beginner's
guide](https://wiki.archlinux.org/index.php/Beginners%27_guide) was very clear
and helpful.

Chose to create a very simple GPT partition scheme using `parted`.

```bash
parted /dev/sda

# Make a GPT partition table
mklabel gpt

# Create a 512GiB EPI System Partition (ESP)
mkpart ESP fat32 1MiB 513MiB

# Make it bootable
set 1 boot on

# Create 2GiB swap
mkpart primary linux-swap 513GiB 2513GiB

# Use the rest for root
mkpart primary ext4 2513GiB 100%
```

This did not work :( the VM was unable to boot up. Tried BIOS/MBR instead.
Created

```bash
parted /dev/sda

# Make a GPT partition table
mklabel msdos

# Create 2GiB swap
mkpart primary linux-swap 1MiB 2GiB

# Use the rest for root
mkpart primary ext4 2GiB 100%

# Make root bootable
set 2 boot on
```

Firewall
--------

[Adapted](/files/archlinux-firewall.txt) an [old project](https://github.com/afreeorange/iptables) 
and things work as expected. Don't forget to [enable the service](https://wiki.archlinux.org/index.php/Iptables#Configuration_and_usage)

    systemctl enable iptables.service

`Package Management` and other stuff
------------------------------------

### Official Repos

`pacman` is meat and potatoes of package management from 'official' sources.
Like CentOS/Red Hat, here's "base", "extra", and "community". Packages get
here in a highly vetted way. [The
wiki](https://wiki.archlinux.org/index.php/Pacman) is a great handbook.

```bash
# Search for stuff
pacman -Ss node

# Install stuff
pacman -S nodejs

# Remove stuff and deps (if not needed by other stuff)
pacman -Rs nodejs

# Clean cache
pacman -Scc

# Upgrade whole system
pacman -Syu
```

### Unofficial Repo

For *everything* else, there's the [Arch User Repository
(AUR)](https://aur.archlinux.org/) which has nearly 30,000 (!) packages. The
usual caveats of non-official sources apply here. To install anything, get a
`PKGBUILD` file for the package, then

```bash
# Make the package with deps and remove them after successful build
makepkg -sr

# Generates a .tar.xz file. Install with pacman
pacman -U package.tar.xz

# Short form
makepkg -sri package.tar.xz
```

**Important**: You can't run any `makepkg` commands as `root`!

And then there's [Yaourt](https://github.com/archlinuxfr/yaourt) which
provides a unified interface to `pacman` and the AUR. Install it like any
other package

```bash
# Be clean
mkdir tmp && cd tmp

# Install package-query as a dep
curl -o package-query https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=package-query
makepkg -sri -p package-query

# Install yaourt
curl -o yaourt https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=yaourt
makepkg -sri -p yaourt
```

All done!

```bash
$ yaourt -Ss pyenv 
aur/pyenv 20151222-1 [installed] (3)
    Simple Python version management
aur/pyenv-virtualenv 20151103-1 [installed] (0)
    pyenv plugin to manage virtualenv (a.k.a. python-virtualenv)
```

### Adding Mirrors

`reflector` will fetch the latest mirrors based on some criteria you provide
it (e.g. I want HTTPS and IPv6 only.) You can [do this
online](https://www.archlinux.org/mirrorlist) as well. 

Other Notes
-----------

*   The Arch Wiki has pretty much everything I needed, written in a clear and
    concise way.
*   `genfstab` won't write the swap partition if you don't `swapon`!
*   Unset `GREP_OPTIONS` if you don't want to go insane with warnings...
*   Export `$EDITOR` when using `visudo` else you'll see [a screen full of
    "EOF"
    messages](https://bbs.archlinux.org/viewtopic.php?pid=796380#p796380).

### `/tmp` size

This is set to a small, fixed size which is [a good
thing](http://superuser.com/a/619398). To install stuff, read the docs about
some way to set the temporary folder. For example, `pyenv` allows you to
export `$TMPDIR` before installation. I use `/var/tmp`

    TMPDIR=/var/tmp pyenv install 3.5.1

However, this can be a little annoying. `systemd` is the one that creates this
mount (since I couldn't find it in `/etc/fstab`... since I *created* it myself
with `genfstab`!) with this

    /usr/lib/systemd/system/tmp.mount

One option would be to rename. A better one would be to simply mask it

    systemctl mask tmp.mount

Setting `/tmp` to a fixed size is still good. But it seems to use half the
RAM; with my VPS box, this is untenable. Since I get tons of storage (and very
little memory), I resorted to creating a 5-10GiB partition just for `/tmp`.

References
----------

* [Using `journalctl`](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs)
* Linode/Arch issues [1](https://bbs.archlinux.org/viewtopic.php?id=184800), [2](https://bbs.archlinux.org/viewtopic.php?id=183911)
