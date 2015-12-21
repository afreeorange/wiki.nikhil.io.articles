You must be superuser for both.

### On Linux

Issue the simple `w` command:

` [root@zenith ~]# w`  
` 15:17:55 up 1 day,  3:35,  1 user,  load average: 0.06, 0.03, 0.01`  
` USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT`  
` root     pts/0    dhcpw80ff96bc.dy 13:24    0.00s  0.04s  0.00s w`  
` tomc     pts/3    dhcpw70gghjbc.dy 13:24    0.00s  0.04s  0.00s w`

Looks like the user **tomc** is logged into terminal 3. Kick the user
out of his session by:

` pkill -9 -u tomc -t pts/3`

### On OS X

Find out the PID of the `loginwindow` process for a given user you want
to kick.

`  ps auxwww|grep loginwindow`

In this case, we want to log out the user `bennifer`

` bennifer    15015   0.0  0.2   281680   7188   ??  Ss   11:06AM   0:00.69 /System/Library/CoreServices/loginwindow.app/Contents/MacOS/loginwindow console`  
` root     15208   0.0  0.0    66152     88 s000  R+    1:30PM   0:00.00 grep loginwindow`

Now kill the PID 15015

` kill -9 15015`
