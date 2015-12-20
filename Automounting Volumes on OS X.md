OS X (client) will only mount external drives on user login (and not
reboot). To correct this,

` sudo defaults write /Library/Preferences/SystemConfiguration/autodiskmount AutomountDisksWithoutUserLogin -bool true`

This creates a file:

` /Library/Preferences/SystemConfiguration/autodiskmount.plist`

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
