Installed [AIDE](http://aide.sourceforge.net/) `v0.13.1-6.el5_8.2` 
on CentOS 5.8 x86\_64

Notes
-----

### Installation

```bash
yum -y install aide  
cp /etc/aide.conf{,.original}
```

### Configuration

The `aide.conf` file has selinux entries which you [might want to
redefine](http://backdrift.org/how-to-fix-aide-lgetfilecon_raw-failed-for-no-data-available-errors?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+Backdrift+%28Backdrift%29)
to prevent "`lgetfilecon_raw failed`" errors. Here are my definitions:

```
ALLXTRAHASHES = sha1+rmd160+sha256+sha512+tiger  
EVERYTHING    = p+i+n+u+g+s+m+c+acl+xattrs+md5+ALLXTRAHASHES  
NORMAL        = p+i+n+u+g+s+m+c+acl+xattrs+md5+rmd160+sha256  
DIR           = p+i+n+u+g+acl+xattrs  
PERMS         = p+i+u+g+acl  
LOG           = p+u+g+i+n+S+acl+xattrs  
LSPP          = p+i+n+u+g+s+m+c+acl+xattrs+md5+sha256  
DATAONLY      = p+n+u+g+s+acl+xattrs+md5+sha256+rmd160+tiger
```

Read the file, determine what you need, and add/adjust entries
accordingly.

### Initialization

```bash
aide --init
```

This creates `/var/lib/aide/aide.db.new.gz`. **Back up this file to a
safe and secure place off this server!** Now rename the file:

```bash
mv /var/lib/aide/{aide.db.new.gz,aide.db.gz}
```

### Checking

```bash
# On the system  
aide --check  
  
# Comparing outside the system  
aide --compare old.db.gz new.db.gz
```
