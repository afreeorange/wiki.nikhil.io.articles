## Maintenance

```bash
# Upgrade PiHole and Gravity lists
pihole -up && pihole -g

# Restart DNS Subsystem
pihole restartdns

# If it complains about an old version of the OS, skip the check
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true sudo -E bash
```

## Logs

```bash
tail -f /var/log/pihole.log
```

## New Lists

To install, go to Settings -> Blocklists and add them there. Then go to Tools -> Update Gravity.

Historical Note: These _used_ to go in `/etc/pihole/adlists.list` but, since PiHole 5+ will use SQLite instead of flat text files, you gotta add these via the UI.

### Regexes

Here's [an overview](https://docs.pi-hole.net/ftldns/regex/overview/). Added [this list](https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list).

### Additional Blocklists

Find a [big list here](https://firebog.net/). Find the "ticked" lists [here](https://v.firebog.net/hosts/lists.php) (these are safe to add and won't cause issues.)

### SmartTV Blocklists

See [this GitHub gist](https://github.com/Perflyst/PiHoleBlocklist/blob/master/SmartTV.txt). I have Samsung TVs, so some caveats apply about blocking domains like `cdn.samsungcloudsolution.com` and (especially) `time.samsungcloudsolution.com`.

### Google AMP 🙄

Add [this list](https://www.github.developerdan.com/hosts/lists/amp-hosts-extended.txt).
