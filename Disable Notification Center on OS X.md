```bash
# Disable permanently
sudo defaults write /System/Library/LaunchAgents/com.apple.notificationcenterui KeepAlive -bool false

# Enable
sudo defaults write /System/Library/LaunchAgents/com.apple.notificationcenterui KeepAlive -bool true

# Kill now
killall NotificationCenter
```