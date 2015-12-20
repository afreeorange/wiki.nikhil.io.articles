Rolls
=====

List installed rolls
--------------------

` rocks list roll`

[Install a roll](http://www.nbcr.net/software/doc/apbs-roll/5.3/adding-the-roll-running.html) (on a running system)
-------------------------------------------------------------------------------------------------------------------

You'll usually get an ISO file. Let's call it rocks-roll.iso

` # Appending clean=1 will remove any older rolls of same name`  
` rocks add roll rocks-roll.iso`  
` `  
` # This should now show the newly added roll`  
` rocks list roll`  
` `  
` # Enable the roll`  
` rocks enable roll sge`  
` `  
` # Rebuild the distro`  
` cd /export/rocks/install`  
` rocks create distro`  
` `  
` # Run the roll; generates a setup shell script`  
` rocks run roll rocks-roll > /tmp/rocks-roll-setup.sh`  
` sh -x /tmp/rocks-roll-setup.sh`

Note: When installing the SGE roll, I had errors about not finding the
"TRANS.TBL" file. Manually copying it
`/export/rocks/install/rolls/sge/5.3/x86_64/RedHat/RPMS` solved the
problem. Strange...

Remove a roll
-------------

` # Disable and remove the roll`  
` rocks disable roll sge`  
` rocks remove roll sge`  
` `  
` # Rebuild the distro`  
` cd /export/rocks/install`  
` rocks create distro`

Nodes
=====

Running a command on all nodes
------------------------------

This is far better than some `bash` cleverness since it's done
*simultaneously* (and not sequentially).

` rocks run host command="yum -y install gcc44*"`

Testing the Kickstart File
--------------------------

` /state/partition1/rocks/install/sbin/kickstart.cgi -c compute-0-1 > compute-0-1.configuration.xml`  
` `  
` # This is better`  
` rocks list host profile compute-0-1 > compute-0-1.xml`

Installing a Node
-----------------

` insert-ethers`  
` # Wait for the node to come up, for ROCKS to assign it a name & IP,`  
` # and for an asterisk like "(*)"`

### Assign a desired name to a node

`insert-ethers` does things very sequentially. If you have another set
of nodes that you want to number differently:

` insert-ethers --rack=1 --rank=14`

This will give your appliance a name like `compute-1-14`

Reinstalling a Node[^1]
-----------------------

Assume that `compute-0-1` is the node under consideration.

### If node is available

On the node to be reinstalled, run

` /boot/kickstart/cluster-kickstart-pxe`

You can also shoot it from the frontend

` shoot-node compute-0-1`

### If node is unavailable

` rocks set host boot compute-0-1 action=install`  
` `  
` # After the node has kickstarted; not doing this will result in infinite install loop`  
` rocks set host boot compute-0-1 action=os`

Removing a Node
---------------

` rocks remove host compute-0-9`

Installing RPMs
---------------

I'm using ROCKS 5.3 on an x86\_64 server.

-   Copy your RPMs to `/export/rocks/install/contrib/5.3/x86_64/RPMS`.
-   Now edit
    `/export/rocks/install/site-profiles/5.3/nodes/extend-compute.xml`
    and add the package name like so:

` `<package>`R`</package>  
` `<package>`libRmath`</package>

-   Then:

` cd /export/rocks/install`  
` rocks create distro`

Change IP address
-----------------

` rocks set host interface ip compute-1-0 iface='eth0' ip='11.255.255.251'`

See all IP and MAC addresses
----------------------------

` rocks list host interface`

Track usage
-----------

` qacct -h`

Removing the "-h" flag will give you the total system usage. This
command has a *lot* of switches and is quite flexible (e.g. per user,
per node, per job, since 8 days)

Miscellaneous
=============

Starting the SSH Agent
----------------------

You may get this error when trying to shoot a node:

` [root@cluster ~]# shoot-node compute-2-1`  
` Shoot Node - version 5.5`  
` Usage:  /opt/rocks/sbin/shoot-node [-h] host ...`  
` Requires ssh-agent to launch`

To start the SSH agent,

` ssh-agent $SHELL`  
` ssh-add`

MySQL
-----

ROCKS runs an alternate MySQL server on port 40000. It's used to
maintain the SGE and Ganglia (amongst other) databases. Here's the full
process:

` /opt/rocks/libexec/mysqld `  
`   --defaults-file=/opt/rocks/etc/my.cnf `  
`   --basedir=/opt/rocks `  
`   --datadir=/var/opt/rocks/mysql `  
`   --user=rocksdb `  
`   --log-error=/var/opt/rocks/mysql/cluster.example.com.err `  
`   --pid-file=/var/opt/rocks/mysql/cluster.example.com.pid `  
`   --socket=/var/opt/rocks/mysql/mysql.sock `  
`   --port=40000`

Luckily, you can turn it on and off like a SysV service:

` service foundation-mysql stop`  
` service foundation-mysql start`

You can also see what's in the `cluster` database by connecting to the
socket specified:

` mysql --socket=/var/opt/rocks/mysql/mysql.sock --user=rocksdb`

Kickstart Params
----------------

View list of params with

` rocks list attr`

Here's how you change the `Kickstart_PublicAddress` attribute

` rocks set attr Kickstart_PublicAddress 129.78.19.15`

Wordpress
---------

I once needed to change the password to the Wordpress administration
page. The ROCKS manual said I should've used `admin` and the cluster's
root password. This didn't work, presumably since I changed the root
password later.

I was able to reset the Wordpress admin password by connecting using
this:

` mysql -uwordpress -p -hlocalhost --socket=/var/opt/rocks/mysql/mysql.sock`

and issuing this:

` UPDATE wordpress.wp_users SET user_pass = md5('PASSWORD') WHERE user_login = 'admin';`

Sources
=======

-   [ROCKS 5.3 Administration
    Guide](http://www.rocksclusters.org/roll-documentation/base/5.3/)
-   [Customizing Configuration of Compute
    Nodes](http://www.rocksclusters.org/roll-documentation/base/5.3/customization-postconfig.html)

References
==========

<references markdown="1">
[^2]

</references>
[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")

[^1]: 

[^2]: [Forcing a Re-install at Next PXE
    Boot](http://www.rocksclusters.org/roll-documentation/base/5.3/x1354.html)
