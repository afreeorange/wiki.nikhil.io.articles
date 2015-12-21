Pre-Install notes
-----------------

The "Very Secure FTP Daemon" is highly configurable package in many
aspects. These include virtual users, SSL transfers, chrooting, and so
on. This guide sets up VSFTP with the following features:

-   A modified FTP root at `<font color="red">/tempftpdir</font>`
-   A sample virtual user called `<font color="red">tempuser</font>`
-   The user will be chrooted to
    `<font color="red">/tempftpdir/tempuser</font>`
-   All transactions will take place over implicit SSL
-   The daemon will run as a standalone service (i.e. will not
    involve xinetd)

Download and install VSFTPD and associated packages
---------------------------------------------------

` yum install vsftpd db4-utils db4`

The last two packages are for the Berkeley DB which PAM uses to look up
virtual users and their passwords.

### Basic configuration

The config file is found at
`<font color="red">/etc/vsftpd/vsftpd.conf</font>` Here are the
pertinent directives which have changed from the original file (which
I'm assuming you will back up before trying this stuff.)

`  anonymous_enable=NO`  
`  dirmessage_enable=NO`  
`  xferlog_file=/var/log/vsftpd.xferlog.tempftpdir.log`  
`  ftpd_banner=Welcome to the CLCG FTP droppoint. Please note that all activity is logged.`  
`  `  
`  # Comment this directive (we will be using another for virtual users)`  
`  pam_service_name=vsftpd`

Managing virtual users
----------------------

### Create the database

You will need to create a text file which has the usernames and
passwords on newlines. E.g.

`  user1`  
`  password1`  
`  user2`  
`  password2`

In this case, I'm going to create a user called **tempuser** with the
password **tempuserpass**. So I create a file called "userlist.txt" with
the following contents:

`  tempuser`  
`  tempuserpass`

Now I can create the database for the PAM using this:

` db_load -T -t hash -f userlist.txt vsftpd-virtual-user.db`

### Configure the home directory

Since the FTP root is `<font color="red">/tempftpdir</font>`, you will
need to add a home directory that `<font color="red">tempuser</font>`
can be chrooted to.

` mkdir /tempftpdir/tempuser`  
` chown -R `[`ftp:ftp`](ftp:ftp)` /tempftpdir`

It should be obvious why `<font color="red">ftp:ftp</font>` owns this
directory; `<font color="red">tempuser</font>` is a *virtual* user and
so does not have any entry in `<font color="red">/etc/passwd</font>`!

### Tell PAM about the database

Head over to `<font color="red">/etc/pam.d/</font>` and create a file
called `<font color="red">vsftpd.withvirtualusers</font>` The filename
can be anything you want. You will need to remember it later!

Add the following to the new file:

`  #%PAM-1.0`  
`  auth       required     pam_userdb.so db=/etc/vsftpd/vsftpd-virtual-user`  
`  account    required     pam_userdb.so db=/etc/vsftpd/vsftpd-virtual-user`  
`  session    required     pam_loginuid.so`

### Configuring VSFTPD to have virtual users

Append the following to the configuration file:

`  # Allow virtual users`  
`  virtual_use_local_privs=YES`  
`  guest_enable=YES`  
`  user_sub_token=$USER`  
`  `  
`  # Change the FTP root `  
`  local_root=/tempftpdir/$USER`  
`  chroot_local_user=YES`  
`  hide_ids=YES`  
`  `  
`  # Use the new file we just created; this is why it was commented earlier!`  
`  pam_service_name=vsftpd.withvirtualusers`  
`  `  
`  # Define passive ports`  
`  pasv_min_port=12000`  
`  pasv_max_port=12003`  
`  `

At this point, you should be ready to start the service. However, you
need to poke a hole in your firewall to allow FTP connections

Configuring IPTABLES to allow FTP
---------------------------------

`  # Allow VSFTPD and associated passive connections`  
`  -A INPUT -m state --state NEW,ESTABLISHED -p tcp --dport 21 -j ACCEPT`  
`  -A INPUT -m state --state NEW,ESTABLISHED -p tcp --dport 12000:12003 -j ACCEPT`

Restart the iptables service (do it properly and use
`<font color="red">iptables-save</font>` and
`<font color="red">iptables-restore</font>`)

Start the service
-----------------

` service vsftpd start`

Check if it's listening to port 21 by trying this...

` netstat -tulpn | grep :21`

... and seeing something like this:

` tcp        0      0 0.0.0.0:21                  0.0.0.0:*                   LISTEN      29790/vsftpd`

Try it out by logging in as `<font color="red">tempuser</font>` with
`<font color="red">tempuserpass</font>`. All should go well :)

Securing VSFTPD with SSL
------------------------

Assuming things have been amazing thus far, you can now SSL enable the
service for logins and transfers.

### Generate an RSA certificate

`  cd /etc/vsftpd`  
`  openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout vsftpd.pem -out vsftpd.pem`

Here's the standard output of this command:

`  Generating a 1024 bit RSA private key`  
`  .......++++++`  
`  ........................................++++++`  
`  writing new private key to '/etc/vsftpd/vsftpd.pem'`  
`  -----`  
`  You are about to be asked to enter information that will be incorporated`  
`  into your certificate request.`  
`  What you are about to enter is what is called a Distinguished Name or a DN.`  
`  There are quite a few fields but you can leave some blank`  
`  For some fields there will be a default value,`  
`  If you enter '.', the field will be left blank.`  
`  -----`  
`  Country Name (2 letter code) [GB]:US`  
`  State or Province Name (full name) [Berkshire]:Iowa`  
`  Locality Name (eg, city) [Newbury]:Iowa City`  
`  Organization Name (eg, company) [My Company Ltd]:Coordinated Laboratory for Computational Genomics`  
`  Organizational Unit Name (eg, section) []:`  
`  Common Name (eg, your name or your server's hostname) []:ftp.example.com`  
`  Email Address []:clcg.it@gmail.com`  
`  `

### SSL-enable VSFTPD

Open up `<font color="red">/etc/vsftpd/vsftpd.conf</font>` and append
the following:

` # Enable SSL`  
` ssl_enable=YES`  
` force_local_data_ssl=YES`  
` force_local_logins_ssl=YES`  
` ssl_tlsv1=YES`  
` ssl_sslv2=NO`  
` ssl_sslv3=NO`  
` rsa_cert_file=/etc/vsftpd/vsftpd.pem`

Your certificate must be in PEM format and include *both* the public and
private keys. Here's an example of what it would look like:

`   -----BEGIN CERTIFICATE-----`  
`   MIIEHTCCA4agAwIBAgIBFjANBgkqhkiG9w0BAQUFADCB1zELMAkGA1UEBhMCVVMx`  
`   DTALBgNVBAgTBElvd2ExEjAQBgNVBAcTCUlvd2EgQ2l0eTEfMB0GA1UEChMWVGhl`  
`   IFVuaXZlcnNpdHkgb2YgSW93YTE+MDwGA1UECxM1VGhlIENvb3JkaW5hdGVkIExh`  
`   Ym9yYXRvcnkgZm9yIENvbXB1dGF0aW9uYWwgR2Vub21pY3MxHjAcBgNVBAMTFXN1`  
`   cHBvcnQuZW5nLnVpb3dhLmVkdTEkMCIGCSqGSIb3DQEJARYVc3VwcG9ydEBlbmcu`  
`   dWlvd2EuZWR1MB4XDTEyMDQyNzE4MDgwMFoXDTIyMDQyNTE4MDgwMFowgb8xCzAJ`  
`   BgNVBAYTAlVTMQ0wCwYDVQQIEwRJb3dhMR8wHQYDVQQKExZUaGUgVW5pdmVyc2l0`  
`   eSBvZiBJb3dhMT4wPAYDVQQLEzVUaGUgQ29vcmRpbmF0ZWQgTGFib3JhdG9yeSBm`  
`   b3IgQ29tcHV0YXRpb25hbCBHZW5vbWljczEaMBgGA1UEAxMRcWluLmVuZy51aW93`  
`   DQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANCYdLYqvSJ4XYiJwl2Bcmc7A/bs`  
`   7RbmeEqmdCBEF/ORZ1Qz3PZkAgDiaSNCdNU8/Z1RMzK8yxQpNTlUO0rTTxmEpCkl`  
`   TLMfvGL5+ef8dry9+dT9VZZTncW9GizQpAlKd9Bix3I7XHN/1MdjWs4zmvjgxARX`  
`   qYGKCLrwBX8VueimV2h1ac50ngAxMHMjQGF6LvdqkGJcwOfg/ArWU5dlu1U9DkAI`  
`   1QvhTqu0+GfvPKbdVp3VdxPJwbCxBFRbiao7QgrnHBSwnhExR6engcPMMcto3b+R`  
`   N+1CJ5RyMmpPVRF9dx+ey3WsZkZBmpCpMMUM8UqMrGyRE36OVnloOPRC3WkCAwEA`  
`   AaOBijCBhzAJBgNVHRMEAjAAMDoGCWCGSAGG+EIBDQQtFitDTENHIEdlbmVyYXRl`  
`   ZCBDZXJ0aWZpY2F0ZSAoT3BlblNTTCB2MC45LjYpMB0GA1UdDgQWBBTKWGxGUSgn`  
`   VW1NrP/04akS1qXtBDAfBgNVHSMEGDAWgBSMMdfJB8d71bkJ62WSA8T2X2GrBTAN`  
`   BgkqhkiG9w0BAQUFAAOBgQAhJpwbcbUS7pLYEahppPin5+6DDtPNqTjVvpHHjpL0`  
`   ZMYEw1E8STzc96FlWO1r/NrHrB1R3qqn5Ptynk7hEH0IrIJjWhv36GCDEvTxpQri`  
`   Kir4Qt3i5hFWiSJnB5/BrRcqnHFYKhcwZvF72Da3B1oxQPI9J0eaAxHLiYfUVfys`  
`   IA==`  
`   -----END CERTIFICATE-----`  
`   -----BEGIN RSA PRIVATE KEY-----`  
`   MIIEpQIBAAKCAQEA0Jh0tiq9InhdiInCXYFyZzsD9uztFuZ4SqZ0IEQX85FnVDPc`  
`   9mQCAOJpI0J01Tz9nVEzMrzLFCk1OVQ7StNPGYSkKSVMsx+8Yvn55/x2vL351P1V`  
`   llOdxb0aLNCkCUp30GLHcjtcc3/Ux2NazjOa+ODEBFepgYoIuvAFfxW56KZXaHVp`  
`   znSeADEwcyNAYXou92qQYlzA5+D8CtZTl2W7VT0OQAjVC+FOq7T4Z+88pt1WndV3`  
`   E8nBsLEEVFuJqjtCCuccFLCeETFHp6eBw8wxy2jdv5E37UInlHIyak9VEX13H57L`  
`   daxmRkGakKkwxQzxSoysbJETfo5WeWg49ELdaQIDAQABAoIBAQCPiakeRXCajKsI`  
`   LouB3naD1JdYzhYjoPn7nGjiXxkAMPkidwHAxnaedy4T5kIRDgQSwfJyInm36NdI`  
`   GM8oIRoYHC7+ZT1PMTJoBU3TNeXa4PtOdfj0FZvGmuatGfEWt5iU27QUxgZLMBaS`  
`   ++8Joqb6k5M3pfbuA0wtf083EN/mz20pIC2q/EEWd7Za8PPb7+t8iWMnLECg5Ulp`  
`   urSj37X9p7M5b3Spf3FksL5YKG/tzvHa7+9hRFScpldt+dKDtN5SKxanlObD10Dv`  
`   RRFqzVwBAoGBAO1YJRnTfYFqCl+Bt311Vtm8TDqYRBNlfaZjX+dG20zEX8LjLbXl`  
`   YumrdD62uii+KhJVzLyIwh++cKB5MCR2PZZwPX+WAJ5vGk7pfHEi+/5dPttM5k0q`  
`   gtirDlwaGJjTa4lVQTP/x0uxGMT03b8+q9qzDcdisx+7EMtS3EZuNWvpAoGBAOD9`  
`   042RpbyTNXfa38Bl2Or2wuuB5Z2T9Zu9+WlnqdwrXNx9ocRl2XyYJVwfL7DY9nSr`  
`   9VJF/aWe5Bmc9/um6/IGry0auw7M4vGBSRNIFFx5411DIheVdsuZPZT+Hop6woUk`  
`   Xr91AtJhpOci4uErmgq9HM8OA3NAWShRLDIDFnWBAoGBAMhGOcBKMrxyQ2CF79SA`  
`   oBHJDzXeaItJd7ZgYnug0co8ZmXoFxlG/6kXkVaeEAXzOUMRfVqVt+DbbOQsftA1`  
`   qhB4k5xGci0+qR9vbB93mtXvzut0P11cAt9bsBlNt/W1aSeQdh2vtncLcFA6I6eN`  
`   9avsrTLS+T1MN4aqW89ejduJAoGBAKrTLa+cOQkvf/YrYZ1z9rmXd7FWI997uoxw`  
`   NhE4mvhGmC/010EFz5ZQ8nS6XPxaDu3Qree0qnv4Ytmrm4EfYJ+XQaPuWr5HA7w3`  
`   3CLepE7+YImr8hOT8OluxRn9w3SC9nQehC27itPvPUQc8cPi1gd3RItU6Xu1DLyW`  
`   vQaP35qBAoGAbfJUtnAk/FuFFQ3bUmOyqC44lURYXqpDBWlTCiA6cXoZ5ciudcW3`  
`   vhIGg1EPda+fliy1LolV1AjG73+vnDgykggu8H1fOKEv7MfvsaLwGUovsz5MeXN+`  
`   xTI8WOKyrAg8ON1DI3uWVhb07HBUGcWS1vUxESXqa9K4+bAbRYFT/9U=`  
`   -----END RSA PRIVATE KEY-----`

### Restart the service

`service vsftpd condrestart`

You can now use an FTP client which supports implicit SSL (Transmit and
Fugu for OS X, FileZilla, etc) and try out the connection. The ordinary
`<font color="red">ftp</font>` won't work and will give you the
following error:

` Connected to ftp.example.com.`  
` 220 Welcome to the FTP droppoint. Please note that all activity is logged.`  
` Name (ftp.example.com:tech): tempuser`  
` 530 Non-anonymous sessions must use encryption.`

If you're a command-line freak like I am, you can always use
`<font color="red">ftp-ssl</font>`

Other notes
-----------

-   The passive ports *must* be defined! Not doing so will result in a
    very long delay between initiating a connection and viewing a
    directory listing.
-   VSFTPD can be configured for anonymous logins whereby a user can
    download a file but not upload anything (like the CentOS mirrors.)
-   Add `listen_address=127.0.0.1` to listen on a given address.

Resources
---------

### Script to add users to database

    #!/bin/bash

    # Add a virtual FTP user to VsFTPd's Berkeley DB
    # by Nikhil Anand,  Mon Feb 22 09:30:45 CST 2010

    # Paths to config path and custom FTP directory
    VSFTPDPATH="/etc/vsftpd"
    DROPPOINT="/data/ftp"

    # Usage information
    if [ $# -ne 2 ]; then
      echo -e "USAGE: `basename $0` <username> <password>\n"
      echo -e "       Using this without the parameters will refresh the DB used by the VsFTP daemon"
      db_load -T -t hash -f $VSFTPDPATH/userlist.txt $VSFTPDPATH/vsftpd-virtual-user.db
      echo -e "       Database refreshed."
      echo -e "       Edit $VSFTPDPATH/userlist.txt to make any changes to virtual users."
      echo -e "       Run this script again when you're done."
      exit
    fi

    # Define username and password 
    USERNAME="$1"
    PASSWORD="$2"

    # Add username and password to flat text file
    echo $USERNAME   >> $VSFTPDPATH/userlist.txt
    echo "$PASSWORD" >> $VSFTPDPATH/userlist.txt
    echo -e "Added ( $USERNAME : $PASSWORD ) to $VSFTPDPATH/userlist.txt"

    # Refresh the Berkeley DB to reflect these additions
    db_load -T -t hash -f $VSFTPDPATH/userlist.txt $VSFTPDPATH/vsftpd-virtual-user.db
    echo -e "Reloaded database \"vsftpd-virtual-user.db\""

    # Create a home directory inside the FTP directory
    mkdir $DROPPOINT/$USERNAME
    chown -R ftp:ftp $DROPPOINT/$USERNAME
    echo -e "Created and changed permissions for $DROPPOINT/$USERNAME"

### Relevant web URIs

-   [Good explanation of Active and Passive mode
    FTP](http://slacksite.com/other/ftp.html)
-   [VSFTPD
    documentation](http://www.redhat.com/docs/manuals/enterprise/RHEL-5-manual/Deployment_Guide-en-US/s2-ftp-servers-vsftpd.html)
