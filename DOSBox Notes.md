I wanted to finish _Chip's Challenge_ after 30 years. Runs on Windows 95. This is written for macOS.

## Installing DOSBox

```bash
brew install dosbox-x
```

Note that this is [DOSBox-X](https://dosbox-x.com/), a [highly enhanced](https://dosbox-x.com/wiki/DOSBox%E2%80%90X%E2%80%99s-Feature-Highlights) version of DOSBox.

Now grab this savepoint (before Internet Explorer, with Sound Blaster 16 Drivers, Standard VGA Drivers).

https://public.nikhil.io/Windows_95.zip

Then grab [the Windows Entertainment Pack (Best Of) via Archive.org](https://archive.org/details/wep_best-of).

## Running Windows

From the extracted folder,

```bash
dosbox-x -conf dosbox.conf
```

This is how you instruct DOSBox to 'boot up' some 'image' (the terms are wrong but you get the idea.)

### Mounting Folders

In `dosbox.conf`, add stuff towards the bottom:

```
[autoexec]
# Lines in this section will be run at startup.
# You can put your MOUNT lines here.
mount x: path/to/some/folder
mount y: Win95
```

### Installing WEP

Once started, you will need to copy the WEP files to `A:` or `B:` and Start -> Run `B:\setup.exe`. You cannot use ISOs. Yay!

### Snapshots

Play around with the menu and save your state to one of many 'slots'.

### TODO

- 3dfx drivers?
