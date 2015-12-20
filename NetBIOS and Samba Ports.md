| Service Name                 | Port | Protocol              |
|------------------------------|------|-----------------------|
| Name Resolution Service      | 137  | UDP                   |
| Datagram Services (browsing) | 138  | UDP                   |
| Session Service              | 139  | TCP                   |
| SMB                          | 445  | TCP Input, UDP Output |

Observations
------------

These are the contents of `/etc/services`:

`netbios-ns 137/udp   # NetBIOS Name Service`  
`netbios-dgm 138/udp  # NetBIOS Datagram Service`  
`netbios-ssn 139/tcp  # NetBIOS Session Service`  
`microsoft-ds 445/tcp # Microsoft Directory Service`

These are the ports that the Samba server listens on. A few things here:

1.  The first three are ports ports used by Windows to 'find' other
    computers (a la "My Network")
2.  The last is for directory services
3.  UDP 137 and 138 are serviced by `nmbd`
4.  TCP 139 and 445 are serviced by `smbd`

Sources
-------

-   [Active Directory and Firewall
    Ports](http://geekswithblogs.net/TSCustomiser/archive/2007/05/09/112357.aspx)

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
