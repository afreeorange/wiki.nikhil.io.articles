[TOC]

Host is **svn.example.com**. There are basically two ways of serving up
a subversion repository. One uses `svnserve`, a lightweight server
(default port 3690). The other is leveraging Apache (`httpd`) via the
WebDAV protocol.

The latter is more complex. But it is extremely flexible in terms of
administration and is the basis for this setup guide. I will be setting
up a single repository at `https://svn.example.com/repository` with SSL,
LDAP-based authentication, and project-specific access control.

Installation
------------

### Getting the RPM

I'm putting the SVN root in /home/svn as well. This can be anywhere.

    yum install subversion mod_dav_svn

This will install Apache and other dependencies as well.

    service httpd start  
    chkconfig --level 345 httpd on

Make sure it's working, and that `iptables` is not causing any issues.
You can use `nmap` for this purpose or just go to
<http://svn.example.com>.

### Preparing `subversion.conf`

Installing the packages will create a new apache configuration directive
in `/etc/httpd/conf.d` called `subversion.conf`. You need to edit this
file to set up the location of the repository.

First uncomment these files if they've not been uncommented:

    LoadModule dav_svn_module     modules/mod_dav_svn.so  
    LoadModule authz_svn_module   modules/mod_authz_svn.so

Define the SVN root:

    <Location /repository>  
            DAV svn  
            SVNPath /home/svn/repository  
    </Location>

Now you can add simple authentication or use LDAP.

Configuration
-------------

### Simple Authentication

This uses basic `htpasswd` based authentication. Passwords may be sent
in the clear if you don't enable SSL. You can also use digest-based
authentication which is slightly more secure.

For this scheme, our `/etc/httpd/conf.d/subversion.conf` file will have
the following directive:

    <Location /repository>  
            DAV svn  
            SVNPath /home/svn/repository  
              
            # Simple authentication  
            AuthType Basic  
            AuthName "SVN Server"  
            AuthUserFile /home/svn/basic-authentication  
            Require valid-user  
    </Location>

Here, we use /home/svn/authorized-users to authenticate. Create this
file and add a user with:

    [root@svn ~]# htpasswd -cm /home/svn/authorized-users testuser  
    New password:   
    Re-type new password:   
    Adding password for user testuser

### Setting up the repository

    svnadmin create /home/svn/repository

Make absolutely sure that Apache owns this directory and its
descendants!

    chown -R apache:apache /home/svn/repository

### Testing the Configuration

At this point, you should have a repo accessible via Apache, with
password sent in clear text (we'll change that). I went to
`http://svn.example.com` and saw the image to the right after entering
my credentials for testuser.

Excellent! Test it now! I tested this config with Eclipse (with the
Subclipse plugin.)

### Securing with SSL

To secure stuff with SSL, generate or use a certificate and enable
Apache with `mod_ssl`. Change `subversion.conf` so that all traffic on
port 80 is redirected to port 443 (which uses the certs we've created.)

     <VirtualHost *:80>  
             ServerName svn.example.com  
             RewriteEngine On  
             RewriteRule .* https://svn.example.com%{REQUEST_URI} [L,R=301]  
     </VirtualHost>

Restart `httpd` and you're good to go!

### LDAP integration

In this example, I will be using **directory.example.com** as the (Open
Directory-based) LDAP provider. Change the basic authentication scheme
to match this:

     # # If using some CA file  
     # LDAPTrustedMode NONE    
     # LDAPTrustedGlobalCert CA_DER /etc/pki/tls/certs/root_ca.crt  
     # LDAPVerifyServerCert off

     # Define the repository location  
     <Location /repository>  
             DAV svn  
             SVNPath /home/svn/repository  
       
             # Integrate with LDAP server  
             AuthType Basic  
             AuthBasicProvider ldap  
             AuthName "SVN Server"  
             AuthzLDAPAuthoritative off  
             AuthLDAPURL "(ldap://directory.example.com/cn=users,dc=directory,dc=example,dc=com?uid?sub)?(objectClass=*)"  
             Require valid-user  
             AuthzSVNAccessFile /home/svn/repository/conf/authz  
     </Location>

**It is important** that you set `AuthBasicProvider ldap`. If not,
Apache will look for a password file and not even bother to authenticate
against your LDAP server. You'll also see something like this when
restarting the `httpd` daemon:

    Invalid command 'AuthLDAPAuthoritative', perhaps misspelled or defined by a module not included in the server configuration

I had terrible luck with setting `AuthzLDAPAuthoritative` to "on". You
can read the Apache `mod_authnz_ldap` page for more information on these
directives. They're quite flexible when configuring multiple
repositories, with respect to user and group access.

Now that you have a single repository, you can fine tune access with the
`AuthzSVNAccessFile` directive. By default, and when you use
`svnadmin create`, you get an `authz` file in your repository's `conf`
folder. In the Apache configuration above, it's the file I've used to
tweak folder access.

Project Management within a Repository
--------------------------------------

### Creating a project

This is very simple. It's vitally important that your project folder
contains three sub-folders: **trunk**, **branches** and **tags**. All
the code you want to check into the repository must be in **trunk**.

#### Step 1: Create the required directory structure

    mkdir -p /tmp/newproject/{trunk,branches,tags}

#### Step 2: Copy/move project files into `trunk`

    cp -R /path/to/project/files/* /tmp/newproject/trunk/

#### Step 3: Perform the first commit

    cd /tmp
    svn import newproject https://svn.example.com/repository/myproject --message "Initial import" --username myuser

Observe that my project is called `newproject` on my local machine but
is `myproject` on the SVN server. You may or may not choose to do this,
but the option is available.

You may get a dialog about the certificate used to secure the
transaction. Accept the key permanently. You will then be required to
supply a password.

#### Step 4: Working with your project

Most typical CVS actions should apply (prefixed with an `svn` of
course.) For example, to check out the project created above.

    svn checkout https://username@svn.example.com/repository/myproject

The Google teems with SVN cheatsheets.

### Modifying Access Control

**Important**: Only root can do this. Talk to your friendly sysadmin for
project-specific access control. By default, your newly created project
will be world accessible (i.e. to *all authenticated* users.)

Here's an example where I created a folder for a rather sinister project
called `thiswillendpoorly` and have given write access only to user
`nanand` and read access to `machrist`. *The leading slash is
important!*

    # Deny world access to repository root (noone needs to get a project listing)  
    [/]  
    * =  
      
    # Allow only Nikhil and Mark to access this terrible project (Mark can only read)  
    [/thiswillendpoorly]  
    nanand = rw  
    machrist = r  
    * =

If you had multiple repositories, you would need to:

*   Change the Apache directive `SVNPath` to `SVNParentPath`
*   Specify the repository in the `authz` file

Here's an example:

    [repository1:/path]  
    user1 = rw  
    user2 = r  
      
    [repository2:/path]  
    * = rw

If you specified a path without specifying the repository, the filter is
applied across *all* repositories! This [is explained
here](http://svnbook.red-bean.com/en/1.0/svn-book.html#svn-ch-6-sect-4.4.2).

Miscellaneous
-------------

### Special note about LDAP groups

You cannot do LDAP group-based authentication in SVN with the `authz`
file. However, I've seen [a python
script](http://www.thoughtspark.org/node/26) which can import LDAP
groups.

### Few pointers on multiple repository configuration

*   If you plan on hosting multiple repositories, you need to change
    `SVNPath` to "SVNParentPath".
*   *Apache will NOT allow you to access the root defined as
    `SVNParentPath`!* You need to create repositories using
    `svnadmin create` and can then access them through
    `http://svn.example.com/{path in SVNParentPath}/{name of repository}`.
    There's [more information in the official handbook about
    this](http://svnbook.red-bean.com/nightly/en/svn.serverconfig.httpd.html#svn.serverconfig.httpd.basic).

### Configuring for use with Self-Signed Certificates

Assuming that your Root CA is called **`root_ca.crt`**. Create and edit
`/etc/sysconfig/servers` to add the following:

    [global]  
    ssl-authority-files = /etc/pki/tls/certs/root_ca.crt

The other option is to use the system-wide keystore at
`/etc/pki/tls/certs/ca-bundle.crt` by appending the ASCII version to the
end of this file.

Resources
---------

*   [mod\_authnz\_ldap directives - Apache page](http://httpd.apache.org/docs/2.1/mod/mod_authnz_ldap.html)
*   [Subversion on CentOS (Wiki)](http://wiki.centos.org/HowTos/Subversion)
*   [Single or multiple repositories?](http://stackoverflow.com/questions/252459/one-svn-repository-or-many/252717#252717)
*   [Version Control with Subversion](http://svnbook.red-bean.com/)
*   [SubVersion with Apache and LDAP integration](http://blogs.open.collab.net/svn/2009/03/subversion-with-apache-and-ldap-updated.html)
*   [Per-directory access control in SVN](http://svnbook.red-bean.com/en/1.0/svn-book.html#svn-ch-6-sect-4.4.2)

### Active Directory Integration

*   [Apache and Subversion authentication with Microsoft Active Directory](http://www.jejik.com/articles/2007/06/apache_and_subversion_authentication_with_microsoft_active_directory/)
*   [mod\_auth\_kerb and mod\_authnz\_ldap bring Apache web apps into the Enterprise](http://www.wadewoolwine.com/2009/01/28/mod_auth_kerb-and-mod_authnz_ldap-bring-apache-web-apps-into-the-enterprise/)
*   [Apache 2.2 – authnz\_ldap – Active Directory](http://michele.pupazzo.org/diary/?p=227)
