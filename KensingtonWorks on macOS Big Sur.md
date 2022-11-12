I'm using an [Expert Mouse Wireless Trackball](https://www.kensington.com/p/products/electronic-control-solutions/trackball-products/expert-mouse-wireless-trackball/) over Bluetooth.

NOTE: I've updated this for macOS Monterey. The instructions should be the same.

### To Uninstall Old Software

1. Go to System Preferences &rarr; Right-click on Kensingtonworks &rarr; Remove
2. Finder &rarr; Go to Folder &rarr; `~/Library/Preferences`
3. Remove `com.kensington.kensingtonworks2.app.plist` and `TbwSettings.json`
4. Run Finder &rarr; Applications &rarr; Utilities &rarr; KensingtonWorks Uninstaller
5. Restart!

### To Fresh Install

1. [Go here](https://www.kensington.com/p/products/ergonomic-desk-accessories/ergonomic-input-devices/expert-mouse-wireless-trackball-1/) and download the latest version (v3.0.4). I cached it [here](https://public.nikhil.io/KensingtonWorks-3.0.4.pkg) as well in case some intrepid PM at Kensington decides to fuck with their website and download links again[^shasum].
2. Run it. It will get stuck at some point.
3. Go to System Preferences &rarr; Security and Privacy &rarr; General and unblock it
4. Install _should_ complete
5. Go to System Preferences &rarr; Security and Privacy &rarr; Privacy &rarr; Accessibility &rarr; check KensingtonWorks Helper (it _should_ be checked after installation.)
6. Restart
7. Launch KensingtonWorks

[^shasum]: Check shasum: `2343262e7647cbed2821cf10ff11cb7c037fbabd2e0282839cd9fc63bd851d8e`
