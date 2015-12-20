Tested on 10.7 (Lion) and 10.6 (Snow Leopard).

Remove all startup items, receipts and the MySQL installation directory:

` sudo rm -rf /Library/StartupItems/MySQL*`  
` sudo rm -rf /Library/PreferencePanes/MySQL*`  
` sudo rm -rf /Library/Receipts/mysql-*`  
` sudo rm -rf /var/db/receipts/com.mysql.*`  
` sudo rm /usr/local/mysql`  
` sudo rm -rf /usr/local/mysql-*`

Remove this in `/etc/hostconfig`

` MYSQLCOM=-NO-`

Done.

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
