[TOC]

Notes from installing ArchLinux on VirtualBox to use as a development machine at work.

Installation
------------

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

Configuration
-------------

### Time and Date

    timedatectl set-timezone America/Chicago

### Hostname

    hostnamectl set-hostname nikhil.io

### Add a normal user

Who will be able to `sudo` and do things and set a password

    useradd -m -g wheel user
    passwd user

Can always change the name of this user later with `usermod`. Then use `visudo` to enable the `wheel` group. You'll see [a lot of artifacts](https://bbs.archlinux.org/viewtopic.php?pid=796380#p796380) if you don't set `EDITOR` first

    EDITOR=vim visudo

### Yoghurt

Edit `/etc/pacman.conf` and add this

    [archlinuxfr]
    SigLevel = Never
    Server = http://repo.archlinux.fr/$arch

Then,

    pacman -Sy yaourt

### Verbose Boot

Modify `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`

### Framebuffer Resolution

Edit `/etc/default/grub`:

    GRUB_GFXMODE=1024x768x32

Then run `grub-mkconfig -o /boot/grub/grub.cfg` and reboot

### Firewall

[Adapted](/files/archlinux-firewall.txt) an [old project](https://github.com/afreeorange/iptables)
and things work as expected. Don't forget to [enable the service](https://wiki.archlinux.org/index.php/Iptables#Configuration_and_usage)

    systemctl enable iptables.service

### Network

The `pacman` update will break networking due [a
bug](https://bugs.archlinux.org/task/41215) that may have been fixed in
`systemd` v228 (as of this writing). Oh well. 
For the interface you see in `ip link` (will start with "`en`")

    systemctl enable dhcpcd@ens4.network

Then enable the appropriate service and restart the node

    systemctl enable systemd-networkd
    reboot

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

VirtualBox Notes
----------------

### VirtualBox Guest Additions

    pacman -S virtualbox-guest-utils \
              virtualbox-guest-modules \
              virtualbox-guest-dkms \
              linux-headers

This is without an LTS kernel since I couldn't be bothered. After installation, enable the service

    systemctl enable vboxservice.service

Edit `/etc/modules-load.d/virtualbox.conf` to add these

    vboxguest
    vboxsf
    vboxvideo

### "virtualbox kernel service is not running"

[Here's the issue](https://bugs.archlinux.org/task/40495). Happened after a system update. Fixed with

    sudo pacman -Su linux-headers

### Resizing

Can only do this with VDIs and not VMDKs. To convert a VMDK (on Windows)

    cd C:\Program Files\Oracle\VirtualBox
    VBoxManage.exe clonehd <path to VMDK> <path to VDI> --format vdi

Then can resize

    VBoxManage.exe modifyhd <path to VDI> --resize 25000

Then boot up VM. `parted` above version 2.4 [doesn't allow you to resize](https://www.gnu.org/software/parted/manual/html_node/Command-explanations.html#Command-explanations) although its `man` page lists it as an option :/ I used GParted instead to fill the rest of the partition and was a happy person. `fdisk` works too.

X11
---

### Installation

    pacman -S xorg-server xorg-xinit xfce4 xfce4-goodies

At this point, running `startxfce4` should show you a desktop. Reboot.

### Starting

    cp /etc/X11/xinit/xinitrc ~/.xinitrc
    echo -e "exec startxfce4" >> ~/.xinitrc

Modify `~/.xinitrc` to remove all the `xterm`, `xclock` and `exec` lines and add this

    exec startxfce4

Now, `startx` should work!

Install some extras

    yaourt -S google-chrome numix-themes numix-circle-icon-theme-git ristretto evince2-light squeeze-git --noconfirm

### Compiz (Maybe)

For Compiz,

    yaourt -S compiz

To run compiz,

    compiz --replace ccp

To get the [Numix theme](https://wiki.archlinux.org/index.php/Compiz_configuration#Window_decoration_themes),

    gsettings set org.gnome.metacity theme theme-name

Add that to "Session and Startup". I had to kill it, _not save the session_, and log out. The default window manager is `xfwm4`.

### The Trash Can

    sudo pacman -S gvfs gamin

### Sound

    pacman -S alsa-firmware alsa-utils
    alsactl init

### Dock

Lots of options, but I like Docky and Plank. Went with Plank. To see preferences,

    plank --preferences

Configuration is kept in `~/.config/plank`.

### NetworkManager

A bit 'heavy' compared to `netctl` but I was tired of fighting with the corporate network.

    pacman -S networkmanager network-manager-applet xfce4-notifyd

Enable the service (else you'll get D-Bus errors when you run `nm-applet`)

    systemctl enable NetworkManager.service
    systemctl start NetworkManager.service

Reboot and log back in. You'll find the network manager in Applications -> Settings -> Network Connections

### "AddScreen/ScreenInit failed for driver 0"

Add `iomem=relaxed` to `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`. Then generate a new grub config and initramfs with

    grub-mkconfig -o /boot/grub/grub.cfg

### Windows Fonts

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

Other Stuff
-----------

### Compacting VDI Images

You'll need [`zerofree`](https://frippery.org/uml/index.html). It works on ext4 filesystems as well. Install it on the VM, then reboot with an Arch LiveCD. Mount the system some place (e.g. `/mnt/vm`) as _read-only_ and zerofree it

    mount -o ro /dev/sda2 /mnt/vm
    zerofree /dev/sda2

Now shutdown the VM (and remove the LiveCD). On the VirtualBox host (mine was Windows)

    cd "C:\Program Files\Oracle\VirtualBox"
    VBoxManage.exe modifyhd c:\path\to\thedisk.vdi --compact

### Dropbox

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

### CA Certificates

Copy certificates in PEM format _and ending with a `.pem` extension_ to `/etc/ssl/certs`. Then, as root, run `update-ca-trust`.

Google Chrome didn't seem to depend on the system store.

### Emoji

Either install `ttf-symbola` or [`emojione-color-font`](https://github.com/eosrei/emojione-color-font)

### Adding Mirrors

`reflector` will fetch the latest mirrors based on some criteria you provide
it (e.g. I want HTTPS and IPv6 only.) You can [do this
online](https://www.archlinux.org/mirrorlist) as well.

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

### Pacman and GPG Proxies

Corporate proxy blocked port 11371 (the default) that Pacman used to get
its keys. Had to modify `/etc/pacman.d/gnupg/gpg.conf` and modify the 
`keyserver` to `hkp://keyserver.kjsl.com:80`
