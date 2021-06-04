PCRE
----

```bash
curl ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.20.tar.gz -O
tar -xvzf pcre-8.20.tar.gz
cd pcre-8.20
./configure --disable-shared --enable-static
make && make install DESTDIR=/src/pcre/pcre-local
```

libjpeg
-------

```bash
cd ..
curl -O http://www.ijg.org/files/jpegsrc.v8c.tar.gz
tar -xvzf jpegsrc.v8c.tar.gz
cd jpeg-8c/
ln -s `which glibtool` ./libtool
./configure –enable-shared && make && sudo make install
```

PHP
---

```bash
# Download PHP
cd ..
curl http://www.opensource.apple.com/source/apache_mod_php/apache_mod_php-53.4/php-5.3.4.tar.bz2 -O
bunzip2 php-5.3.4.tar.bz2
tar -xvf php-5.3.4.tar
cd php-5.3.4

# Now change "libiconv" to "iconv"
vim ext/iconv/iconv.c

# Configure
./configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info 
--disable-dependency-tracking --sysconfdir=/private/etc --with-apxs2=/usr/sbin/apxs 
--enable-cli --with-config-file-path=/etc --with-libxml-dir=/usr --with-openssl=/usr 
--with-kerberos=/usr --with-zlib=/usr --enable-bcmath --with-bz2=/usr --enable-calendar 
--with-curl=/usr --enable-exif --enable-ftp --with-gd --with-jpeg-dir=/usr/local/lib 
--with-png-dir=/usr/X11R6 --with-freetype-dir=/usr/X11R6 --with-xpm-dir=/usr/X11R6 
--with-ldap=/usr --with-ldap-sasl=/usr --enable-mbstring --enable-mbregex 
--with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd 
--with-mysql-sock=/tmp/mysql.sock --with-iodbc=/usr --enable-shmop --with-snmp=/usr 
--enable-soap --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm 
--with-xmlrpc --with-iconv-dir=/usr --with-xsl=/usr --with-pcre-regex=/src/pcre/pcre-local/usr/local

# Compile (this takes time)
export EXTRA_CFLAGS="-lresolv"
make

# Back up the php.ini file and install
cp /etc/php.ini /etc/php.ini.bak
make install
apachectl restart
```

Sources
-------

* <http://www.gen-x-design.com/archives/recompiling-php-5-3-on-snow-leopard-with-freetype-support/>




