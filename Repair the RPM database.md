I had a problem with running a `yum update` on some package. The process
would sleep indefinitely. A clue that something was wrong with the RPM
database (at `/var/lib/rpm`) was that running `rpm -qa` would hang as
well.

The solution is to rebuild the RPM database. In short:

` # Backups!`  
` tar -czvf /tmp/rpm.tgz /var/lib/rpm`  
` `  
` # Remove the database locks`  
` rm -f /var/lib/rpm/__db*`  
` `  
` # Rebuild!`  
` rpm -vv --rebuilddb`

That should do it.

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
