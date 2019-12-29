Saving here [via](https://feeding.cloud.geek.nz/posts/encoding-wifi-access-point-passwords-qr-code/)

```bash
# On macOS. Note to self: use pwgen to generate secure passwords.
brew install qrencode pwgen

# Set the SSID
SSID="My Home Wifi"

# Set a looooong password for the AP
pwgen -s 63 > "${SSID}.password.txt"

# Put it all together
qrencode \
  --size=15 \
  --output=${SSID}.png \
  "WIFI:T:WPA;S:${SSID};P:$(cat "${SSID}.password.txt");;"
```

Goes without saying that should be displayed in a location that's not visible from, say, a window. On iOS, the camera app automagically recognizes the resulting QR code and asks if you'd like to join the network. 
