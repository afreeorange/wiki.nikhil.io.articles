Pre-flight
----------

Generated random passwords for backup user with this small script:

    #!/bin/bash
    for i in $(seq 20); do
      password=$(tr -dc A-Za-z0-9_ < /dev/urandom | head -c 20)
      echo -e "GRANT LOCK TABLES, SELECT, RELOAD, SUPER ON *.* TO 'backupuser'@'backup.example.com' IDENTIFIED BY '$password';"
    done

CentOS/RHEL
-----------

### On client

The MySQL user's homedir is `/var/lib/mysql`. This is nice.

      cd /var/lib/mysql
      mkdir .ssh
      chmod 700 .ssh
      chown mysql:mysql .ssh
      cd .ssh
      touch authorized_keys
      chmod 600 authorized_keys
      chown mysql:mysql authorized_keys
      echo "ssh-rsa {public key here} root@backup.example.com" >> authorized_keys

1.  Tested connection by issuing `ssh mysql@client.example.com` on
    backup server.
2.  Logged onto MySQL server on client and issued one of the GRANT
    statements above.
3.  Created list of databases to back up.
4.  Saved password in book.

### On backup server

Created a backup config for client in
`/etc/mysql-zrm/backupset-client/mysql-zrm.conf`. Here's a sample
compressed, encrypted, selective backup that is rotated every 30 days.
The passphrase is not the SQL backup password.

    # MySQL connection params
    host=pbackup.example.com
    user=backupuser
    password=xAhganJiakstZ
    databases="drupal_ddd redmine"

    # Host connection params and options
    copy-plugin="/usr/share/mysql-zrm/plugins/ssh-copy.pl"

    # Backup params
    backup-mode=logical
    compress=1
    encrypt=1
    passfile=/etc/mysql-zrm/backupset-pdb/passphrase

    # Retention policy
    retention-policy="30D"

OS X Server
-----------

### On Client

1.  Created a *local* account called `mysqlbackupuser` in System
    Preferences &gt; Accounts with password.
2.  Added user to System Preferences &gt; Sharing &gt; Remote Login
3.  SSH-ed in as user, then

<!-- -->

      cd ~
      mkdir .ssh
      chmod 700 .ssh
      chown mysql:mysql .ssh
      cd .ssh
      touch authorized_keys
      chmod 600 authorized_keys
      chown mysql:mysql authorized_keys
      echo "ssh-rsa {public key here} root@backup.example.com" >> authorized_keys

1.  Tested connection from backup server
2.  Logged on to MySQL instance to add `backupuser` GRANT statement.

### On backup server

Config was the same as CentOS/RHEL, except that I added the `ssh-user`
param:

` ssh-user=mysqlbackupuser`

Problems
--------

MySQL v5.0.95 is not compiled with support for profiling (the "SHOW
PROFILES" command.) This is the `--enable-profiling` tag at
compile-time. Strange, considering that the previous version (v5.0.77)
had support for profiling. The upshot is that you can't select
individual databases to be backed up if you're running 5.0.95. The
Webtatic and Remi repos have MySQL 5.5+ if you're desperate.

