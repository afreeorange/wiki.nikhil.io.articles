Maintenance
-----------

```bash
# Upgrade PiHole and Gravity lists
pihole -up && pihole -g

# Restart DNS Subsystem
pihole restartdns
```

Logs
----

```bash
tail -f /var/log/pihole.log
```

New [Regexes](https://docs.pi-hole.net/ftldns/regex/overview/)
--------------------------------------------------------------

Added [this](https://github.com/mmotti/pihole-regex/blob/master/regex.list) to `/etc/pihole/regex.list`. Just looks for tracking keywords.

```
# https://github.com/mmotti/pihole-regex/blob/master/regex.list
^(.+[-_.])??adse?rv(er?|ice)?s?[0-9]*[-.]
^(.+[-_.])??m?ad[sxv]?[0-9]*[-_.]
^(.+[-_.])??xn--
^adim(age|g)s?[0-9]*[-_.]
^adtrack(er|ing)?[0-9]*[-.]
^advert(s|is(ing|ements?))?[0-9]*[-_.]
^aff(iliat(es?|ion))?[-.]
^analytics?[-.]
^banners?[-.]
^beacons?[0-9]*[-.]
^count(ers?)?[0-9]*[-.]
^pixels?[-.]
^stat(s|istics)?[0-9]*[-.]
^telemetry[-.]
^track(ers?|ing)?[0-9]*[-.]
^traff(ic)?[-.]

# Instart Logic
(.*)\.g00\.(.*).
```

Then restart via `sudo service pihole-FTL restart`

New Blocklists
--------------

Find a [big list here](https://firebog.net/). Find the "ticked" lists [here](https://v.firebog.net/hosts/lists.php) (these are safe to add and won't cause issues.) 

To install, go to Settings -> Blocklists and add them there. These _used_ to go in `/etc/pihole/adlists.list` but adding these via the UI is better since PiHole 5 will use SQLite instead of flat text files.

### SmartTV Blocklists

See [this GitHub gist](https://github.com/Perflyst/PiHoleBlocklist/blob/master/SmartTV.txt). I have Samsung TVs, so some caveats apply about blocking domains like `cdn.samsungcloudsolution.com` and (especially) `time.samsungcloudsolution.com`.

### Google AMP 🙄

Add [this list](https://www.github.developerdan.com/hosts/lists/amp-hosts-extended.txt)
