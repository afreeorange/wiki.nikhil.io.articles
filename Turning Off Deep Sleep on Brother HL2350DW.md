Because it's annoying as fuck if you set up wireless printing. I'm assuming a PM at Brother thought this would save the planet _and_ get them a promotion 🙄

- Go to **General Setup** -> <kbd>OK</kbd>
- Scroll with arrow keys to **Ecology** -> <kbd>OK</kbd>
- Scroll with arrow keys to **Sleep Time** -> <kbd>OK</kbd>
- Set it to 0 minutes -> <kbd>OK</kbd>
- Now you have to hold _both_ the back button and down button to see the 'secret' **Deep Sleep** option. Turn it off and you'll see "Accepted".
- Hit <kbd>OK</kbd> the appropriate number of times to get back to the main menu

### Update for macOS

This _still_ does not clear deep sleep _sometimes_. There's apparently a maintenance mode option. I read through their [troubleshooting guide](https://help.brother-usa.com/app/answers/detail/a_id/151825/~/unable-to-print-after-the-machine-has-entered-deep-sleep---windows#Recommeded). Then disabled IPv6 and things _appear_ to work. **Update** It's still shit. Nothing works with this freaking thing, not even the latest firmware update. I had to resort to 👇

### Bash Script for Unifi Controller

👉 This only works if you use Unifi gear. Here's a small `bash` script that 'kicks' the printer every hour or so. This is the equivalent of clicking "Reconnect" from the Unifi Controller UI. Unbelievable. There's a PM at Brother who deserves at least one terrible week.

```bash
#!/bin/bash

# REFERENCES:
# https://ubntwiki.com/products/software/unifi-controller/api

set -euo pipefail

USERNAME="your_username"
PASSWORD="lol"
CONTROLLER_URI="https://192.168.1.3:8443"
COOKIE_PATH="./cookie.txt"
CURL_COMMAND="curl -s -S --cookie ${COOKIE_PATH} --cookie-jar ${COOKIE_PATH} --insecure "

MAC_ADDRESS="aa:bb:cc:dd:ee:ff"

login() {
    ${CURL_COMMAND} \
        -H "Content-Type: application/json" \
        -X POST \
        -d "{\"password\":\"$PASSWORD\",\"username\":\"$USERNAME\"}" \
        $CONTROLLER_URI/api/login >/dev/null
}

logout() {
    ${CURL_COMMAND} $CONTROLLER_URI/logout
}

kick_the_fucking_printer() {
    response=$(${CURL_COMMAND} \
        -H "Content-Type: application/json" \
        -X POST \
        --data-binary "{\"cmd\":\"kick-sta\",\"mac\":\"$MAC_ADDRESS\"}" \
        $CONTROLLER_URI'/api/s/default/cmd/stamgr' \
        --compressed)

    # This doesn't return any data, really. Else, you can use jq to parse it.
    echo "----------"
    echo "$response"
    echo "----------"
}

login
kick_the_fucking_printer
logout
```
