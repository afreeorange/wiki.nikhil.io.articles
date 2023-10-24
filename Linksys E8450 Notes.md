I was on v1.1 of the firmware and decided to upgrade it. If you did this, you won't be able to dismiss the setup wizard like OpenWRT wants you to do 🤦‍♀️ _You must pick a simple admin password to finish the setup!_ Something like `Leboeuf1!` and not `j!V4BE@4kuK%*H4Z`.

Else, when the wizard logs you out, you will see a lovely "LOGIN FAILED" message when you enter the password you entered during setup 🤦‍♀️

Once the wizard is done, upgrade to v1.2 of the firmware. I downloaded `FW_E8450_1.2.00.360516_prod_signed.img` from [this page](https://www.linksys.com/support-article?articleNum=317332).

Then follow [these instructions](https://github.com/dangowrt/owrt-ubi-installer#installing-openwrt). I used these files:

- `openwrt-23.05.0-mediatek-mt7622-linksys_e8450-ubi-initramfs-recovery-installer_signed.itb` - Once you boot up with this firmware, any changes you make are _not permanent_. You must use the `sysupgrade` image below.
- `openwrt-23.05.0-mediatek-mt7622-linksys_e8450-ubi-squashfs-sysupgrade.itb`

Then follow some post-install instructions (like setting the Wifi country).

