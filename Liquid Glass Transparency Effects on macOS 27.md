These don't really appear to work for me but jotting here anyway. ([Cached from this post](https://www.reddit.com/r/MacOS/comments/1wir6oa/heres_how_to_bring_back_more_transparency_in/)).

![](https://public.nikhil.io/wiki.nikhil.io/liquid-glass.jpeg)

### Softer scroll edge

```bash
# System-wide
defaults write -g MPV8 -bool false

# Per-app example: Mail
defaults write com.apple.mail MPV8 -bool false

# Restore system default
defaults delete -g MPV8

# Restore Mail default
defaults delete com.apple.mail MPV8
```

### Fully transparent

```bash
# System-wide
defaults write -g NSScrollPocketEnabled -bool false

# Per-app example: Mail
defaults write com.apple.mail NSScrollPocketEnabled -bool false

# Restore system default
defaults delete -g NSScrollPocketEnabled

# Restore Mail default
defaults delete com.apple.mail NSScrollPocketEnabled
```

Restart affected apps, or reboot Mac.

