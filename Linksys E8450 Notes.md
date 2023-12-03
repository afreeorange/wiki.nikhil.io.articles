### OpenWRT Installation Notes

I was on v1.1 of the firmware and decided to upgrade it. If you did this, you won't be able to dismiss the setup wizard like OpenWRT wants you to do 🤦‍♀️ _You must pick a simple admin password to finish the setup!_ Something like `Leboeuf1!` and not `j!V4BE@4kuK%*H4Z`.

Else, when the wizard logs you out, you will see a lovely "LOGIN FAILED" message when you enter the password you entered during setup 🤦‍♀️

Once the wizard is done, upgrade to v1.2 of the firmware. I downloaded `FW_E8450_1.2.00.360516_prod_signed.img` from [this page](https://www.linksys.com/support-article?articleNum=317332).

Then follow [these instructions](https://github.com/dangowrt/owrt-ubi-installer#installing-openwrt). I used these files:

- `openwrt-23.05.0-mediatek-mt7622-linksys_e8450-ubi-initramfs-recovery-installer_signed.itb` - Once you boot up with this firmware, any changes you make are _not permanent_. You must use the `sysupgrade` image below.
- `openwrt-23.05.0-mediatek-mt7622-linksys_e8450-ubi-squashfs-sysupgrade.itb`

Then follow some post-install instructions (like setting the Wifi country).

### Enabling WPS on OpenWRT

**NOTE**: Doing this fried my router! I wasn't even able to power it on :( Keeping these notes here anyway. Just using stock Linksys firmware now.

Look at [this page](https://blog.sakuragawa.moe/connect-wi-fi-in-a-wps-push-button-way-with-openwrt/) and [the official wiki doc](https://openwrt.org/docs/guide-user/network/wifi/basic#wps_options). In short, SSH into the router and

```bash
opkg update
opkg remove wpad-mini
opkg install wpad hostapd-utils
```

Then edit `/etc/config/wireless` and find your Wifi SSID. Add this option:

```
wifi-iface 'Home WiFi'
  ...
  option wps_pushbutton '1'
```

Restart the network via `service network restart`. Then you can set up WPS pairing by running

```
hostapd_cli wps_pbc
```
