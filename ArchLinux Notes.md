[TOC]

Installation
------------

[Downloaded](https://www.archlinux.org/download/) the ISO (`2015.12.01`) and
set up an "Other 64-bit" VM in VMWware Fusion 8 on OS X El Capitan. Wired 
networking was working at bootup. The Arch [beginner's guide](https://wiki.archlinux.org/index.php/Beginners%27_guide) was very clear and helpful.

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

This did not work :( the VM was unable to boot up. Tried BIOS/MBR instead. Created 

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

[Adapted](/files/archlinux-firewall.txt) an [old 
project](https://github.com/afreeorange/iptables) and things work as expected.

Package Management
------------------

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

Some basic packages

```bash
pacman -S ack bash-completion git htop libxml2 \
          libxslt nginx openssh php-fpm postfix \
          dovecot python rsync supervisor vim wget
```

### Unofficial Repo

For *everything* else, there's the [Arch User 
Repository (AUR)](https://aur.archlinux.org/) which has nearly 30,000 (!) 
packages. The usual caveats of non-official sources apply here. To install 
anything, get a `PKGBUILD` file for the package, then

```bash
# Make the package with deps and remove them after successful build
makepkg -sr

# Generates a .tar.xz file. Install with pacman
pacman -U package.tar.xz

# Short form
makepkg -sri package.tar.xz
```

**Important**: You can't run any `makepkg` commands as `root`!

And then there's [Yaourt](https://github.com/archlinuxfr/yaourt) which provides
a unified interface to `pacman` and the AUR. Install it like any other
package

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

Other Notes
-----------

*   The Arch Wiki has pretty much everything I needed, written in a clear and
    concise way. 
*   `genfstab` won't write the swap partition if you don't `swapon`!
*   Unset `GREP_OPTIONS` if you don't want to go insane with warnings...
*   Export `$EDITOR` when using `visudo` else you'll see
    [a screen full of "EOF" messages](https://bbs.archlinux.org/viewtopic.php?pid=796380#p796380).

### `/tmp` size

This is set to a small, fixed size which is [a good thing](http://superuser.com/a/619398).
To install stuff, read the docs about some way to set the temporary folder. For
example, `pyenv` allows you to export `$TMPDIR` before installation. I use
`/var/tmp`

    TMPDIR=/var/tmp pyenv install 3.5.1

However, this can be a little annoying. `systemd` is the one that creates this 
mount (since I couldn't find it in `/etc/fstab`... since I *created* it 
myself with `genfstab`!) with this

    /usr/lib/systemd/system/tmp.mount

One option would be to rename. A better one would be to simply mask it

    systemctl mask tmp.mount

Setting `/tmp` to a fixed size is still good. But it seems to use half the
RAM; with my VPS box, this is untenable. Since I get tons of storage (and
very little memory), I resorted to creating a 5-10GiB partition just
for `/tmp`.
