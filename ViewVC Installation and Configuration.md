ViewVC presents you with a nice browser interface to SVN and CVS
repositories. This is an installation document for **ViewVC v1.1.5** on
**RHEL 5.4**. It depends on Python, but you should have that installed
on RHEL5 (for `yum`) anyway.

Download and Install
--------------------

[Get it here](http://www.viewvc.org/download.html) and extract the
tarball to a temporary location. Within, you will find `viewvc-install`,
a shell script which makes installation a breeze.

    sudo ./viewvc-install  

I chose the installation directory at `/var/www/html/viewvc`. The
default is `/usr/local/viewvc-1.1.5`. Adapt to your liking.

Configuration
-------------

All configuration is done via `/var/www/html/viewvc/viewvc.conf`. I'm
now going to set up ViewVC to see SVN and CVS repos on the same host.

*   I have an SVN repository in `/home/svn/repository` (served over
    Apache as `svn.example.com/repository`)
*   The CVS repository is located in `/home/cvs`.

### Setting up the both Repositories

These are the only lines different from the original file:

    [general]  
    cvs_roots = cvsroot: /home/cvs  
    svn_roots = repository: /home/svn/repository  
    root_parents = /home/svn : svn,  
                   /home/cvs : cvs  
    address = support@example.com  
      
    [utilities]  
    svn = /usr/bin/svn  
    diff = /usr/bin/diff  
      
    [options]  
    authorizer = svnauthz  
    use_rcsparse = 1  
      
    [authz-svnauthz]  
    authzfile = /home/svn/repository/conf/authz

The configuration file is pretty nicely documented. The configuration
above respects the `authz` file for an SVN repo. Any user not authorized
to 'see' a project will not be able to do so, even via the web. Yay!

### Telling Apache about ViewVC and securing the connection

In `/etc/httpd/conf.d`, create a config directive called `viewvc.conf`
and add the following:

    ScriptAlias /viewvc "/var/www/html/viewvc/bin/cgi/viewvc.cgi"  
    ScriptAlias /query "/var/www/html/viewvc/bin/cgi/query.cgi"

You will also want to prevent any external access. We will use LDAP and
SSL to secure access via Apache. The directives are similar to an SVN
setup.

    <Location /viewvc>  
        # Integrate with LDAP server  
        AuthType Basic  
        AuthBasicProvider ldap  
        AuthName "CBCB SVN Server"  
        AuthzLDAPAuthoritative off  
        AuthLDAPURL "(ldap://directory.example.com/cn=users,dc=directory,dc=example,dc=com?uid?sub)?(objectClass=*)"  
        Require valid-user  
    </Location>

Other Niceties
--------------

### CVSGraph

[Displays a very nice graph](http://www.akhphd.au.dk/~bertho/cvsgraph/)
of revisions and branches for a given project. You can find it [at
EPEL](http://fedoraproject.org/wiki/EPEL/FAQ#howtouse). Installation
goes:

    yum install cvsgraph --enablerepo=epel  

The binary is `/usr/bin/cvsgraph`. Configuration is at
`/etc/cvsgraph.conf`. To configure ViewVC with it, append the following:

    Under [utilities]  
      cvsgraph = /usr/bin/cvsgraph  
      
    Under [options]  
      use_cvsgraph = 1  
      cvsgraph_conf = /etc/cvsgraph.conf  
    

I've included the config file since I enabled anti-aliasing.

### Pygments

This is a python package that prettifies source code views. You will
need the `python-setuptools` RPM. This installs (among other things) the
`easy_install` command which can be used to install Pygments and other
python packages.

    yum install python-setuptools  
    easy_install pygments

And you're set! Your code displays should be spiffy now.

### Custom Highlighting

I wanted to make `.module` for Drupal highlight as PHP files. For this,
I enabled/added the following to `viewvc.conf`:

    mime_types_files = mimetypes.conf,  
                     /etc/mime.types

This ensures that ViewVC is able to pass on to Pygments (the
highlighter) correct, comprehensive and custom MIME types. Now to enable
my custom config for `.module` files, I just add this to
`mimetypes.conf` (in the same folder):

    application/x-httpd-php         php module

Errors
------

### Rlog Errors

You might see this for a CVS project:

    Error: Rlog output ended early. Expected RCS file...

To remedy this, you need to:

1.  Install RCS (`yum install rcs`)
2.  Tell Apache to serve them properly
3.  Make sure that the user Apache runs as can run
    `rlog /home/cvs/filename.ext,v`

For the latter, edit `httpd.conf` and find the following line:

    IndexIgnore .??* *~ *# HEADER* README* RCS CVS *,v *,t

Try removing the `CVS` and `*,v` entries.

### If they're still not solved

Just use the "experimental" `rcsparse` Python module. [From this mailing
list](http://markmail.org/message/l6tujq2zvh4z5wzt#query:%22Rlog%20output%20ended%20early.%20Expected%20RCS%20file%22+page:1+mid:5zkeunyusumiu3rj+state:results):

> `use_rcsparse` tells ViewVC to use an "experimental" Python-based RCS file  
> parser library instead of parsing the output of the RCS toolchain  
> binaries.  It mostly works, but doesn't support some things (like  
> keyword substitution).  

Find `use_rcsparse` under **[options]** and set it to `1`.
