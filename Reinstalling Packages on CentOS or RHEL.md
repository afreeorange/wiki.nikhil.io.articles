Installed package lists are kept in `/var/log/rpmpkgs`.

` cp /var/log/rpmpkgs /backups/packages`

Reinstall with

` yum install $(sed 's/-[0-9].*.rpm$//g' /backups/packages | tr '\n' ' ')`

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
