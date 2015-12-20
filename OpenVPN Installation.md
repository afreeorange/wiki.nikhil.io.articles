Latest version of OpenVPN is v2.1.1. Installing this on an i386 CentOS
5.5 box (vpn.example.com)

Step 1: Install OpenVPN
-----------------------

` yum info openvpn`

This shows version 2.1.1, so we'll go ahead an install it.

` yum install openvpn`

Step 2: Set up certificate params
---------------------------------

Navigate to `/etc/openvpn/easy-rsa/2.0`. This folder contains a bunch of
scripts to make certificate and access management very easy. You will
edit the `vars` file. Here are the pertinent changes I've made:

` # Why not keep all keys in one place?`  
` export KEY_DIR="/etc/pki/tls/openvpn"`  
`   `  
` # Changed this to one year from ten years`  
` export KEY_EXPIRE=365`  
` `  
` # Organizational Unit Name will be asked during certificate generation`  
` export KEY_COUNTRY="US"`  
` export KEY_PROVINCE="IA"`  
` export KEY_CITY="Iowa City"`  
` export KEY_ORG="The University of Iowa"`  
` export KEY_EMAIL="support@vpn.example.com"`

Step 3: Make yourself a Certificate Authority (CA)
--------------------------------------------------

You'll need this step to sign server and client certificate signing
requests (CSR's). Here's a transcript of what I did (still at
`/etc/openvpn/easy-rsa/2.0`):

` [root@bouncer 2.0]# source ./vars `  
` NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/pki/tls/openvpn`  
` [root@bouncer 2.0]# ./clean-all `  
` [root@bouncer 2.0]# ./build-ca `  
` Generating a 1024 bit RSA private key`  
` ...........................................++++++`  
` ...................................................++++++`  
` writing new private key to 'ca.key'`  
` -----`  
` You are about to be asked to enter information that will be incorporated`  
` into your certificate request.`  
` What you are about to enter is what is called a Distinguished Name or a DN.`  
` There are quite a few fields but you can leave some blank`  
` For some fields there will be a default value,`  
` If you enter '.', the field will be left blank.`  
` -----`  
` Country Name (2 letter code) [US]:`  
` State or Province Name (full name) [IA]:`  
` Locality Name (eg, city) [Iowa City]:`  
` Organization Name (eg, company) [The University of Iowa]:`  
` Organizational Unit Name (eg, section) []:`**`The` `Center` `for`
`Bioinformatics` `and` `Computational` `Biology`**  
` Common Name (eg, your name or your server's hostname) [The University of Iowa CA]:`**`vpn.example.com`**  
` Name []:          `  
` Email Address [support@vpn.example.com]:`

As you can see, I mostly hit return for most values since I pre-defined
them in `vars`

Step 4: Create a server certificate
-----------------------------------

For this, you'll simply execute `build-key-server`. Here's a transcript:

` [root@bouncer 2.0]# ./build-key-server vpn.example.com`  
` Generating a 1024 bit RSA private key`  
` .............................++++++`  
` ...++++++`  
` writing new private key to 'vpn.example.com.key'`  
` -----`  
` You are about to be asked to enter information that will be incorporated`  
` into your certificate request.`  
` What you are about to enter is what is called a Distinguished Name or a DN.`  
` There are quite a few fields but you can leave some blank`  
` For some fields there will be a default value,`  
` If you enter '.', the field will be left blank.`  
` -----`  
` Country Name (2 letter code) [US]:`  
` State or Province Name (full name) [IA]:`  
` Locality Name (eg, city) [Iowa City]:`  
` Organization Name (eg, company) [The University of Iowa]:`  
` Organizational Unit Name (eg, section) []:The Center for Bioinformatics and Computational Biology`  
` Common Name (eg, your name or your server's hostname) [vpn.example.com]:`  
` Name []:`  
` Email Address [support@vpn.example.com]:`  
` `  
` Please enter the following 'extra' attributes`  
` to be sent with your certificate request`  
` A challenge password []:`  
` An optional company name []:`  
` Using configuration from /etc/openvpn/easy-rsa/2.0/openssl.cnf`  
` Check that the request matches the signature`  
` Signature ok`  
` The Subject's Distinguished Name is as follows`  
` countryName           :PRINTABLE:'US'`  
` stateOrProvinceName   :PRINTABLE:'IA'`  
` localityName          :PRINTABLE:'Iowa City'`  
` organizationName      :PRINTABLE:'The University of Iowa'`  
` organizationalUnitName:PRINTABLE:'The Center for Bioinformatics and Computational Biology'`  
` commonName            :PRINTABLE:'vpn.example.com'`  
` emailAddress          :IA5STRING:'support@vpn.example.com'`  
` Certificate is to be certified until Jun 12 15:15:45 2020 GMT (3650 days)`  
` Sign the certificate? [y/n]:y`  
` `  
` 1 out of 1 certificate requests certified, commit? [y/n]y`  
` Write out database with 1 new entries`  
` Data Base Updated`

Step 5: Create one or more client certificates
----------------------------------------------

Run `build-key` for each client. For instance, I'm going to generate a
certificate for myself (user ID `nanand`):

` [root@bouncer 2.0]# source ./vars`  
` [root@bouncer 2.0]# ./build-key nanand`  
` Generating a 1024 bit RSA private key`  
` ....++++++`  
` ......++++++`  
` writing new private key to 'nanand.key'`  
` -----`  
` You are about to be asked to enter information that will be incorporated`  
` into your certificate request.`  
` What you are about to enter is what is called a Distinguished Name or a DN.`  
` There are quite a few fields but you can leave some blank`  
` For some fields there will be a default value,`  
` If you enter '.', the field will be left blank.`  
` -----`  
` Country Name (2 letter code) [US]:`  
` State or Province Name (full name) [IA]:`  
` Locality Name (eg, city) [Iowa City]:`  
` Organization Name (eg, company) [The University of Iowa]:`  
` Organizational Unit Name (eg, section) []:`**`The` `Center` `for`
`Bioinformatics` `and` `Computational` `Biology`**  
` Common Name (eg, your name or your server's hostname) [nanand]:`  
` Name []:`**`Nikhil` `Anand`**  
` Email Address [support@vpn.example.com]:`**`nanand@vpn.example.com`**  
` `  
` Please enter the following 'extra' attributes`  
` to be sent with your certificate request`  
` A challenge password []:`  
` An optional company name []:`  
` Using configuration from /etc/openvpn/easy-rsa/2.0/openssl.cnf`  
` Check that the request matches the signature`  
` Signature ok`  
` The Subject's Distinguished Name is as follows`  
` countryName           :PRINTABLE:'US'`  
` stateOrProvinceName   :PRINTABLE:'IA'`  
` localityName          :PRINTABLE:'Iowa City'`  
` organizationName      :PRINTABLE:'The University of Iowa'`  
` organizationalUnitName:PRINTABLE:'The Center for Bioinformatics and Computational Biology'`  
` commonName            :PRINTABLE:'vpn.example.com'`  
` name                  :PRINTABLE:'Nikhil Anand'`  
` emailAddress          :IA5STRING:'nanand@vpn.example.com'`  
` Certificate is to be certified until Jun 12 15:22:26 2020 GMT (3650 days)`  
` Sign the certificate? [y/n]:`**`y`**  
` `  
` 1 out of 1 certificate requests certified, commit? [y/n]`**`y`**  
` Write out database with 1 new entries`  
` Data Base Updated`

It's **bloody important** to just hit `Enter` when you're asked the
Common Name (CN) for the client certificate! The OpenVPN server assigns
client IPs based on the CN in the client certificate. If you set the CN
of the *clients* the same as that of the *server*, each client will get
the same IP.

Step 6: Configure the server
----------------------------

Copy `/etc/openvpn/sample-config-files/server.conf` to its top-level
directory, `/etc/openvpn`

` cp /etc/openvpn/sample-config-files/server.conf /etc/openvpn/`

Edit the file to configure your server. Here are the pertinent lines I
changed:

` ca /etc/pki/tls/openvpn/ca.crt`  
` cert /etc/pki/tls/openvpn/vpn.example.com.crt`  
` key /etc/pki/tls/openvpn/vpn.example.com.key`  
` dh /etc/pki/tls/openvpn/dh1024.pem`  
` `  
` server 10.34.2.0 255.255.255.0`  
` `  
` push "dhcp-option DNS 19.67.17.20"`  
` push "dhcp-option DNS 19.67.17.21"`  
` push "dhcp-option DOMAIN vpn.example.com"`  
` `  
` max-clients 50`

Step 7: Enable the Management Interface
---------------------------------------

This provides you a realtime 'view' of connected users (in quotes since
it's all textual.) You can also kill user sessions. To enable this, you
need a line like
`management (management IP address) (management port number) (text file for management password)`
in `/etc/openvpn/server.conf`.

Here's what I added:

` management 127.0.0.1 7896 management-password.txt`

Make sure that the management password file is readable only by a
superuser!

### Using the interface

Simple. Telnet. Here's a transcript that shows me using the interface to
figure out who's connected using the **status** command:

` [root@bouncer ~]# telnet 127.0.0.1 7896`  
` Trying 127.0.0.1...`  
` Connected to localhost.localdomain (127.0.0.1).`  
` Escape character is '^]'.`  
` ENTER PASSWORD:`**`*********`**  
` SUCCESS: password is correct`  
` >INFO:OpenVPN Management Interface Version 1 -- type 'help' for more info`  
` `**`status`**  
` OpenVPN CLIENT LIST`  
` Updated,Tue Nov 30 05:18:54 2010`  
` Common Name,Real Address,Bytes Received,Bytes Sent,Connected Since`  
` nikhil,173.27.1.155:58008,37145,35061,Tue Nov 30 05:17:54 2010`  
` ROUTING TABLE`  
` Virtual Address,Common Name,Real Address,Last Ref`  
` 10.34.0.6,nikhil,173.27.1.155:58008,Tue Nov 30 05:18:54 2010`  
` GLOBAL STATS`  
` Max bcast/mcast queue length,0`  
` END`  
` quit`  
` Connection closed by foreign host.`

Start the Server
----------------

` service openvpn start`

Resources
---------

### Differences between `tun` and `tap`

Now the mechanism by which OpenVPN connects /dev/tun on computer A with
/dev/tun on computer B is this: It simply creates an encrypted UDP
connection over the internet between A and B and forwards traffic
between /dev/tun on A with /dev/tun on B. Because of the clever way in
which the tun and tap drivers were designed, it is possible for a
program running entirely in user-space to effect this link, allowing
OpenVPN to be a portable cross-platform daemon (like SSH), rather than
an OS-specific kernel module (like IPSec).

The difference between a tun and tap device is this: a tun device is a
virtual IP point-to-point device and a tap device is a virtual ethernet
device. So getting back to the "long cable" analogy, using a tun device
would be like having a T1 cable connecting the computers and using a tap
device would be like having an ethernet network connecting the two
computers

### Clients

-   [Windows-only Management Interface in .NET
    2.0](http://www.mertech.com.au/default.aspx)
-   [TunTun, a GUI applet for Gnome](http://code.google.com/p/tuntun/)

### Presentations

-   [OpenVPN Presentation by Adrian
    Bridgett](http://www.archive.org/details/HantsLUG_openvpn)
-   [Presentation by James
    Yonan](http://openvpn.net/papers/BLUG-talk/2.html)

### Other

-   [OpenVPN Open Source information
    page](http://openvpn.net/index.php/open-source.html)
-   [OpenVPN MiniFAQ](http://workaround.org/openvpn-faq)

### Building the OpenVPN RPM yourself

Download the tarball from [OpenVPN's Community Downloads
page](http://openvpn.net/index.php/open-source/downloads.html).

` wget `[`http://openvpn.net/release/openvpn-2.1.1.tar.gz`](http://openvpn.net/release/openvpn-2.1.1.tar.gz)

You can now use `rpmbuild` to create a nice RPM for future deployment.
But it does have some dependencies:

` yum install gcc glibc-devel openssl openssl-devel lzo-devel pam-devel pkcs11-helper-devel`

**Note**: `lzo-devel` and `pkcs11-helper-devel` might not be available
on a stock system. I like EPEL and [RPM's by
Remi](http://blog.famillecollet.com/en). The `yum` command will work
flawlessly if you install these repos.

` wget `[`http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-3.noarch.rpm`](http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-3.noarch.rpm)  
` wget `[`http://rpms.famillecollet.com/enterprise/remi-release-5.rpm`](http://rpms.famillecollet.com/enterprise/remi-release-5.rpm)  
` rpm -Uvh remi-release-5*.rpm epel-release-5*.rpm`

You should now be ready to build the RPM.

` rpmbuild -tb openvpn-2.1.1.tar.gz`

The finished RPM will be located in:

` /usr/src/redhat/RPMS/i386/openvpn-2.1.1-1.i386.rpm`

You could store this someplace else if you wanted to install it on other
32-bit systems.

### Firewall Stuff & Typical Problems

Make sure that this is set to 1 in `/etc/sysctl.conf`

` net.ipv4.ip_forward = 1`

For immediate effect,

` echo 1 > /proc/sys/net/ipv4/ip_forward  `

Here are some basic, highly liberal rules:

<bash> $IPTABLES -A INPUT -i tun+ -m state --state NEW -j ACCEPT
$IPTABLES -A INPUT -p udp --dport 1194 -m state --state NEW -j ACCEPT
$IPTABLES -A FORWARD -i tun+ -m state --state NEW -j ACCEPT $IPTABLES -A
POSTROUTING -s 192.168.0.0/16 -t nat -j MASQUERADE $IPTABLES -A
POSTROUTING -s 172.16.0.0/12 -t nat -j MASQUERADE $IPTABLES -A
POSTROUTING -s 10.0.0.0/8 -t nat -j MASQUERADE </bash>

If you use puppet to manage `sysctl.conf` (like I do), you can edit
`/etc/init.d/openvpn` and uncomment the line that enables IP forwarding
immediately.

-   [Very basic
    rules](http://serverfault.com/questions/39307/linux-ip-forwarding-for-openvpn-correct-firewall-setup)
-   [A more complicated
    setup](http://blogs.encodo.ch/news/view_article.php?id=196)

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
