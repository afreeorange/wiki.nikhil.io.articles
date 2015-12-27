[Some background](http://www.tuaw.com/2012/02/14/mac-os-xs-quarantineevents-keeps-a-log-of-all-your-downloads/).

Here's how you see the files:

      sqlite3 \
      ~/Library/Preferences/com.apple.LaunchServices.QuarantineEvents* \
      'select LSQuarantineDataURLString from LSQuarantineEvent'

I have a crontab entry that removes the downloads every night.

    * 0 * * * sqlite3 ~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV* 'delete from LSQuarantineEvent'
