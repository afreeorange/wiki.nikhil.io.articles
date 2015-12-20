    # Disable permanently
    sudo defaults write /System/Library/LaunchAgents/com.apple.notificationcenterui KeepAlive -bool false

    # Enable
    sudo defaults write /System/Library/LaunchAgents/com.apple.notificationcenterui KeepAlive -bool true

    # Kill now
    killall NotificationCenter

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: OS X](Category:_OS_X "wikilink")
