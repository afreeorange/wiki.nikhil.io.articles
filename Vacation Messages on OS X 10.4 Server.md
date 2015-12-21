The Situation
-------------

-   Mac OS X 10.4.11 Server
    -   Postfix, Cyrus, Dovecot
-   Couldn't put vacation in user's homedir
-   Couldn't find vacation binary
-   Other sieve programs only worked with SquirrelMail, only with user
    homedirs
-   Roundcube plugin needed FTP access... to put stuff in homedirs
-   Apple's Mail Server documentation to set up sieve is crap

The Resolution
--------------

Use Cyrus IMAP's sieve to set up a script that auto replies. It needs to
be set up first.

` sudo mkdir /usr/sieve`  
` sudo chown cyrusimap:mail /usr/sieve`

Add yourself to user group that can administer sieve by editing
`/etc/imapd.conf`:

` admins: cyrusimap, admin`

Now add `sieve 2000/tcp #Sieve mail filtering` to `/etc/services`.
Restart mail:

` sudo serveradmin stop mail`  
` sudo serveradmin start mail`

Server should now be listening on port 2000:

` netstat -an | grep 2000`  
` tcp4       0      0  *.2000                 *.*                    LISTEN`  
` tcp6       0      0  *.2000                 *.*                    LISTEN`

Make a vacation script. It can be anywhere. Here's the example from [the
Mail Server admin
guide](http://manuals.info.apple.com/en/Mail_Service_v10.4.pdf):

` #--------`  
` # This is a sample script for vacation rules.`  
` # Read the comments following the pound/hash to find out`  
` # what the script is doing.`  
` #---------`  
` #`  
` `  
` # Make sure the vacation extension is used.`  
` require "vacation";`  
` `  
` # Define the script as a vacation script`  
` vacation`  
` `  
` # Send the vacation response to any given sender only once every seven days no matter how many messages are sent from him.`  
` :days 7`  
` `  
` #For every message sent to these addresses`  
` :addresses ["stone-purchasing@zeus.eng.uiowa.edu"]`  
` `  
` # Make a message with the following subject`  
` :subject "Out of Office Reply"`  
` `  
` # And make the body of the message the following`  
` "Test auto reply";`  
` `  
` # End of Script`

Enter the sieve shell

` /usr/bin/cyrus/test/sieveshell --user=testuser --authname=admin localhost`  
` connecting to localhost`  
` Please enter your password: `  
` > put vacation.msg`  
` > activate vacation.msg `  
` > quit`

This creates `/usr/sieve/t/testuser`. Type 'help' for help. Use
'deactivate' to remove all scripts. Test. Fin.
