When things are OK
------------------

```bash
# Simply remove and install the package  
yum -y remove PACKAGE  
yum -y install PACKAGE
```

When things get bad
-------------------

```bash
# Use rpm to force removal and reinstall with yum  
rpm -e --nodeps PACKAGE  
yum -y install PACKAGE
```

When things go very South
-------------------------

```bash
# Trick yum into thinking that the package  
# doesn't exist in the RPM database  
rpm -e --nodeps --justdb PACKAGE  
yum install PACKAGE
```

N.B. The last method doesn't actually remove any files.

Other notes
-----------

Files staged for install when performing a `yum update` are kept in
`/var/cache/yum/`. This is useful in [certain situations](http://fir3net.com/Redhat-/-Fedora/yum-update-shows-module-object-has-no-attribute-httpshandler-error.html).
