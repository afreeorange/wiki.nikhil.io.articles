Maintenance
===========

```bash
# Upgrade PiHole and Gravity lists
pihole -up && pihole -g

# Restart DNS Subsystem
pihole restartdns
```

New [Regexes](https://docs.pi-hole.net/ftldns/regex/overview/)
==============================================================

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

# AMP, which no one asked for
# Full list: https://www.github.developerdan.com/hosts/lists/amp-hosts-extended.txt
^(.+\.)?amp\.cloudflare\.com$
^(.+\.)?cdn\.ampproject\.org$
```

Then restart via `sudo service pihole-FTL restart`