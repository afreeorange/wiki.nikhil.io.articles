A Gryphon being a glitchy, dinky, _slow_ overpriced piece of shit sold to innocent and beleagured parents I have the displeasure of interacting with. Some fucking notes.

## Setup

Remove all connections.

Reset to factory settings by holding the reset button in the back for 15 seconds until the lights go off and then go solid.

Now wait till the LED blinks. The router's booting up. Wait for about 90 seconds.

_It will blink when there's no internet_. When it starts blinking, launch the stupid app.

**NOTE** 👉 You will spend a lot of time trying to pair the device with the app. You _must make sure that the app has access to Bluetooth!_ It won't ask you to do this until you tap the PPPoe/WAN setup.

Now plug a cable into the INTERNET port from the LAN port of your ISP's modem. When the blinking stops, the Gryphon has been issued an IP.

**NOTE** 👉 _This does not mean internet access_! In my case I had to switch from one LAN port to another to get the app to declare that the thing was ONLINE.

Blinking can _also_ mean that the dumb thing is ready to pair with the app. Pairing might appear stuck. Quit the app and reload it. It will be OK.

Make sure the thing is _blinking continuously_ before you plug into the INTERNET port. The app is checking for two things:

- Connection to the Gryphon, _and_
- Connection to the Internet

If the latter fails, it will blink and the app will throw an error.

## Other

You will have to restart and reset constantly until things fucking work. Try a different port on your ISP's router.

Ethernet and `192.168.1.1` will give you nothing useful. All configuration is done via the app.

On the app, requests will time out and the app will crash constantly. Please expect this and have some hard liquor ready.

You'll see a default Wifi network of `GryphonHomexxx`. The default password is `ghome1234`

It's hard to tell what's going on. You only have one stupid LED that uses two words to communicate fifty ideas.
