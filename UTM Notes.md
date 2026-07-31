Most of this was so I could play some vintage games in DOS/Windows.

Here's a summary.

| OS          | Notes                              |
| ----------- | ---------------------------------- |
| DOS         | Use DOSBox-X                       |
| Windows 3.1 |                                    |
| Windows 95  |                                    |
| Windows XP  |                                    |
| Windows 7   |                                    |
| Windows 10  |                                    |
| Windows 11  | Will never try this piece of shit. |

----

UTM being a QEMU-based hypervisor for M-series Macs.

## A .UTM File/Folder

Simple stuff. An XML file with the VM's settings, a QCOW file for storage, and a PNG for the last saved screenshot of the VM (assuming it's not headless.)

```
Mac OS 9.2.2.utm/
                .
                ├── config.plist
                ├── Data
                │   └── 66A59C7F-0075-43A5-8AE8-77A88A1D0DDC.qcow2
                └── screenshot.png
```

TODO: That QCow file can be snapshotted. How?

## Windows

A PITA as usual. Well, just Windows 7. Microsoft has

### Versions

**Windows 7 64-bit**

[Use this image](). The key is `H7TYK-QK3RD-YYU45-ZZZCD-3VMBM`

### Sharing

PITA. You'll need to make your own SPICE tools ISO [from the source](https://www.spice-space.org/download.html). Get these two:

- [SPICE Guest Tools](https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe)
- [SPICE WebDAV Daemon](https://www.spice-space.org/download/windows/spice-webdavd/)

Put them in a folder and use [Keka](https://www.keka.io/en/) to make an ISO (drag and drop the folder onto Keka after selecting "ISO" in the dropdpwn) that you mount into the VM. Install both. Reboot. And you'll see the DAV folder on port 9783 in My Computer when you share a folder.

---

## Errors

### "Failed to access data from shortcut."

Do this, then restart UTM.

```bash
rm ~/Library/Containers/com.utmapp.UTM/.com.apple.containermanagerd.metadata.plist
```
