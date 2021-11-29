👉 Not a fresh install: things didn't work out of the box when I upgraded. 

I'm using an [Expert Mouse Wireless Trackball](https://www.kensington.com/p/products/electronic-control-solutions/trackball-products/expert-mouse-wireless-trackball/) over Bluetooth.

1. Go to System Preferences &rarr; Right-click on Kensingtonworks &rarr; Remove
2. Finder &rarr; Go to Folder &rarr; `~/Library/Preferences`
3. Remove `com.kensington.kensingtonworks2.app.plist` and `TbwSettings.json`
4. Run Finder &rarr; Applications &rarr; Utilities &rarr; KensingtonWorks Uninstaller
5. Restart!
6. [Go here](https://www.kensington.com/p/products/electronic-control-solutions/trackball-products/expert-mouse-wireless-trackball/) and download the latest version (v2.2.8) 
7. Run it. It will get stuck at some point.
8. Go to System Preferences &rarr; Security and Privacy &rarr; General and unblock it
9. Install _should_ complete
10. Go to System Preferences &rarr; Security and Privacy &rarr; Privacy &rarr; Accessibility &rarr; check KensingtonWorks Helper (it _should_ be checked after installation.)
11. Restart
12. Launch KensingtonWorks
