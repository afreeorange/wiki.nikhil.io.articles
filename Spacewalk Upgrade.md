Written for Spacewalk 1.7

Back up everything
------------------

    cd /root/
    mkdir spacewalk-backup; cd spacewalk-backup
    tar -czvf etc.sysconfig.rhn.tgz /etc/sysconfig/rhn 
    tar -czvf etc.rhn.tgz /etc/rhn
    tar -czvf etc.jabberd.tgz /etc/jabberd

### Backing up the Oracle XE instance

Use this script

    DB_LOCATION="/usr/lib/oracle"
    BACKUP_LOCATION="/backups/oracle"

    SERVICE=$(which service)
    RSYNC=$(which rsync)

    # Take the database offline
    $SERVICE oracle-xe stop

    # Rsync to backup location
    $RSYNC -av --progress --whole-file $DB_LOCATION/ $BACKUP_LOCATION/

    # Start the database
    $SERVICE oracle-xe start

    # Log this
    echo -e `date` > $BACKUP_LOCATION/lastbackup.log

### Backing up the PostgreSQL instance

` pg_dump -U spaceuser -d spaceschema --password | bzip2 > spaceschema.backup.bz2`

Set up the new Spacewalk Repository
-----------------------------------

` rpm -Uvh `[`http://spacewalk.redhat.com/yum/1.5/RHEL/5/x86_64/spacewalk-repo-1.5-1.el5.noarch.rpm`](http://spacewalk.redhat.com/yum/1.5/RHEL/5/x86_64/spacewalk-repo-1.5-1.el5.noarch.rpm)

This will modify /etc/yum.repos.d/spacewalk.repo to point at version
1.5.

Upgrade
-------

    # Stop the Spacewalk service
    /usr/sbin/spacewalk-service stop
    /usr/sbin/spacewalk-service status

    # Upgrade Spacewalk
    yum upgrade --enablerepo=epel --enablerepo=spacewalk

    # Make sure OracleXE is running
    /usr/sbin/oracle-xe status

    # Upgrade the schema
    /usr/bin/spacewalk-schema-upgrade

    # Update the configuration (use XE for SID, spacewalk/spacewalk for user/pass)
    spacewalk-setup --disconnected --upgrade

    # Enable monitoring and monitoring scout 
    /usr/share/spacewalk/setup/upgrade/rhn-enable-monitoring.pl --enable-scout

Cleaning up
-----------

    # Clean RHN cache
    rm -rf /var/cache/rhn/satsync
    rm -rf /var/cache/rhn/xml-*

    # Clean jabberd
    rm -f /var/lib/jabberd/db/*

Restart Spacewalk
-----------------

` /usr/sbin/spacewalk-service start`
