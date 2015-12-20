-   Version installed was v2.1.1
-   **Server** is server.example.com (64-bit CentOS 6.2)
-   **Client** is client.example.com (64-bit CentOS 5.8)

Preparing the server
--------------------

` yum -y install mysql MySQL-zrm`

Make a copy of, and start editing the ZRM config file.

` cp /etc/mysql-zrm/mysql-zrm.conf{,.original}`

### Global Params

I edit `/etc/mysql-zrm/mysql-zrm.conf` to change the default backup
destination.

` destination=/backups/databases`  
` encrypt-plugin="/usr/share/mysql-zrm/plugins/encrypt.pl"`  
` decrypt-option="-d"`  
` mailto="support@example.com"`  
` mail-policy=always`  
` html-report-directory=/var/www/html/reports/`  
` html-reports=backup-status-info,backup-method-info,backup-performance-info,backup-retention-info`

There are [plenty of other options
available](http://wiki.zmanda.com/index.php/How_do_I_configure_MySQL_ZRM).

### Creating a client backupset

Create a directory with whatever you want to call your backupset:

` mkdir /etc/mysql-zrm/`**`backupset-client.example.com`**

Now create the client config file

` vim /etc/mysql-zrm/`**`backupset-client.example.com`**`/mysql-zrm.conf`

I'm going to try encrypted backups. You can create a passphrase file
called `.passphrase` file in the backupset directory or define your own
name for it. I'm trying the latter option.

I added this for basic trial run:

` # MySQL connection params`  
` host=client.example.com`  
` user=backupser`  
` password=Pas5a7hasd`  
` `  
` # Host connection params and options`  
` copy-plugin="/usr/share/mysql-zrm/plugins/ssh-copy.pl"`  
` `  
` # Backup params`  
` backup-mode=logical`  
` compress=1`  
` encrypt=1`  
` passfile=/etc/mysql-zrm/backupset-client.example.com/passphrase`  
` `  
` # Retention policy`  
` retention-policy="10D"`

Preparing the Client
--------------------

### Backup User

Make sure that your backup user [has the appropriate
privileges](http://wiki.zmanda.com/index.php/Do_I_need_to_make_changes_to_MySQL_database_configuration%3F#MySQL_backup_user).
We'll be using "support":

` GRANT LOCK TABLES, SELECT, RELOAD, SUPER on *.* TO 'backupuser'@'backupserver.tld' IDENTIFIED BY 'pgubSfHpm';`

### Keyless access

ZRM will use the "mysql" user to connect. Therefore, we need to set up
keyless SSH access. On CentOS, the mysql user's homedir is
`/var/lib/mysql`. So:

` cd /var/lib/mysql`  
` mkdir .ssh`  
` chmod 700 .ssh`  
` chown mysql:mysql .ssh`  
` cd .ssh`  
` touch authorized_keys`  
` chmod 600 authorized_keys`  
` chown mysql:mysql authorized_keys`  
` echo "ssh-rsa {your public key} root@server.example.com" >> authorized_keys`

Now place the key of the backup server in the authorized keys file and
test a connection:

` ssh mysql@client.example.com`

If you log in, you're good to go! You can alternately create another
user if you don't want to muck with the MySQL user.

Testing
-------

With the server and client configured, try this on the server.

` mysql-zrm-scheduler --now --backup-set backupset-client.example.com --backup-level 0`

If all goes well, you should see a timestamped backup folder in the
backup destination (`/backups/databases` in our case) with a compressed
file called `backup-data`

### Extracting the compressed backup

**Don't use gzip!** Rather:

` mysql-zrm --action extract-backup --source-directory /backups/databases/backupset-client.example.com/20061012232713`

If your backup is encrypted, decrypt it first:

` gpg --decrypt --output test.gz /backups/databases/backupset-client.example.com/20061012232713/backup-data`

It's always advisable to use the ZRM restore feature first.

### Generating reports

` mysql-zrm-reporter`  
` `  
` # To see a particular backup set`  
` mysql-zrm-reporter --where backup-set=backupset-client.example.com`  
` `  
` # Generate HTML reports (output is in backup destination dir)`  
` mysql-zrm-reporter --destination /backups/databases --type html --output report.html`

There are a [a lot of other options
available](http://wiki.zmanda.com/index.php/Mysql-zrm-reporter). I
wanted global reports and set this in my crontab:

` 30 4 * * *  cd /var/www/html/zrm && /usr/bin/mysql-zrm-reporter --destination /backups/databases/ --type html --output index.html --show backup-status-info,backup-method-info,backup-performance-info,backup-retention-info  >> /dev/null 2>&1`

### Validation

` mysql-zrm --action verify-backup --backup-set backupset-client.example.com`

### [Scheduling](http://wiki.zmanda.com/index.php/Mysql-zrm-scheduler)

` # Schedule daily at 1:30PM`  
` mysql-zrm-scheduler --add --interval daily --start 13:30 --backup-set backupset-client.example.com`  
` `  
` # View all scheduled jobs`  
` mysql-zrm-scheduler --query`  
` `  
` # Remove the scheduled job`  
` mysql-zrm-scheduler --delete -interval daily --start 13:30 --backup-set backupset-client.example.com`

This is added to crontab.

### Rotating Backups

Set the retention policy for a given backup set and run this against
your backup destination:

` mysql-zrm --action purge --destination /backups/databases/`

See your crontab. All purge jobs run at 4:00AM (granularity is one day.)

Plugins
-------

These are installed in `/usr/share/mysql-zrm/plugins`. Some good stuff!

Errors and Miscellaneous
------------------------

### Ubuntu

The MySQL user's homedir on Ubuntu is `/nonexistent`. While this makes
sense, you may want to change this for keyless access.

### mysql-bin.00000x: Cannot stat: No such file or directory

Make sure that this is uncommented in your global or client configs:

`  mysql-binlog-path="/var/log/mysql"`

Make sure that the `log-bin` directive in your client config is set.

### ERROR 1381 (HY000) at line 1: You are not using binary logging

Edit `my.cnf` to add this line:

` log-bin=/var/log/mysql/mysql-bin.log`

Make sure that the folder exists and has the right permissions:

` mkdir /var/log/mysql`  
` chown mysql:mysql /var/log/mysql`  
` service mysqld restart`

### Lost connection to MySQL server at 'reading initial communication packet', system error: 111

This is because:

-   Your firewall might be blocking connections
-   The MySQL user specified doesn't have the right privileges (e.g.
    does 'backupuser'@'backupserver.tld' have privileges? Or did you
    set 'backupser'@'localhost'?)
-   The `bindaddress` in `my.cnf` must be set to your target's IP and
    *not* `127.0.0.1`
-   The `skip-networking` option is set in `my.cnf`

Sources
-------

-   [MySQL ZRM Users
    Manual](http://wiki.zmanda.com/index.php/Zmanda_Recovery_Manager_for_MySQL_Users_Manual)

[Category:Installation Notes](Category:Installation_Notes "wikilink")
[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
