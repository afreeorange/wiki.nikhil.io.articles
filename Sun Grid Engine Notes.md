Nodes
=====

All nodes (and info)
--------------------

` qhost`  
` `  
` # Get a list of all compute nodes`  
` qhost | grep compute | awk '{print $1}'`

Queues
======

Viewing all Queues
------------------

` qstat -f`

Enabling, Disabling, Clearing
-----------------------------

### Single Queue

` # Enabling a queue`  
` qmod -e all.q@compute-1-3.local`  
` `  
` # Disabling a queue`  
` qmod -d all.q@compute-1-3.local`  
` `  
` # Clearing error state`  
` qmod -c all.q@compute-1-3.local`

### All Queues

` # Enable all.q on all nodes`  
` qmod -e '*'`  
` `  
` # Disable all.q on all nodes`  
` qmod -d '*'`  
` `  
` # Clear all.q on all nodes`  
` qmod -c '*'`

You can also disable a single queue on all nodes by replacing ` '*'`
with the name of the queue.

Jobs
====

All jobs currently running or in queue
--------------------------------------

` qstat -u "*"`

Details on a particular job
---------------------------

` # E.g. Job ID is 568798`  
` qstat -explain c -j 568798`

Miscellanea
===========

SGE Variables
-------------

All GE vars are in `/opt/gridengine/default/common`

Backups
-------

` # Use this to `[`back`
`up`](http://download.oracle.com/docs/cd/E19957-01/820-0698/gemnb/index.html)` (prefer tarball)`  
` inst_sge -bup`

` # Use this `[`to`
`restore`](http://download.oracle.com/docs/cd/E19957-01/820-0698/gemkx/index.html)  
` inst_sge -rst`

Restarting the SGE Service
--------------------------

` service sgemaster.$(hostname -s) stop`

Sources
-------

-   [How to Administer Sun Grid
    Engine](http://biowiki.org/HowToAdministerSunGridEngine)

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
