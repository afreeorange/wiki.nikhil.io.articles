Installation
------------

-   Need to install **mysql-server**
-   Run the server (Otherwise:
    `Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock'`)

Configuration
-------------

-   Initialize server installation by creating root password  
    `mysqladmin -u root password ''new_root_password''`
-   Config file for **mysqld** is at `/etc/my.cnf`
    -   Databases are stored in `/var/lib/mysql`
    -   Socket at `/var/lib/mysql/mysql.sock`

Administration
--------------

-   Use **mysqladmin** to change passwords
    -   E.g.
        `mysqladmin -u testuser -p ''user_old_password'' password ''user_new_password''`
-   The other way is to edit the **mysql** database itself
    -   `update user set password=PASSWORD("''user_new_password''") where User='testuser';`

### Reset `root` password

MySQL Safe Mode is how the root user's password may be reset in case
someone misplaces/forgets it. Needless to say, this must be run as
**root** on the server.

1.  Stop the daemon  
    `/etc/init.d/mysqld stop`
2.  Start Safe Mode  
    `mysqld_safe --skip-grant-tables &`
3.  Connect as root  
    `mysql -u root`
4.  Play with the **mysql** database  
    `mysql> use mysql;`
5.  Update the **user** table  
    ` mysql> update user set password=PASSWORD("''new_root_password''") where User='root';`
6.  Refresh privileges  
    ` mysql> flush privileges;`
7.  Quit and restart the **mysqld** daemon (which was in safe mode)  
    `mysql> quit`  
    `/etc/init.d/mysqld stop`  
    `/etc/init.d/mysqld start`

Resources
---------

-   [Recovering from a corrupted MySQL install due to a dying
    HDD](http://blog.nickj.org/2010/03/16/recovering-from-a-corrupted-mysql-install-due-to-a-dying-hard-disk/)

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
