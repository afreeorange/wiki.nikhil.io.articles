I installed Redmine v0.9.4 on a 32-bit CentOS 5.5 box. The following
versions were used:

-   Ruby v1.8.7
-   RubyGems 1.3.7
-   Rails v2.3.5
-   Passenger v2.2.14

You'll need a MySQL database ready, so set it up and make a note of the
username and password. There are quite a few steps involved, so let's
go!

Pre-Flight
----------

`yum install httpd-devel openssl-devel zlib-devel gcc gcc-c++ curl-devel expat-devel gettext-devel ImageMagick ImageMagick-devel mysql-devel`

You *may* have issues with `mysql-devel`. In this case,

` yum remove mysql`  
` yum install mysql mysql-devel mysql-server`

Step 1: Compile and Install Ruby
--------------------------------

The FTP site is at <ftp://ftp.ruby-lang.org//pub/ruby/1.8/>

` wget `[`ftp://ftp.ruby-lang.org//pub/ruby/1.8/ruby-1.8.7.tar.gz`](ftp://ftp.ruby-lang.org//pub/ruby/1.8/ruby-1.8.7.tar.gz)  
` tar -xzvf ruby-1.8.7.tar.gz`  
` sudo ./configure --enable-shared --enable-pthread`  
` make`  
` make install`

The weird thing is that you now have to tell Ruby about `zlibs` and
re-compile. In the compilation folder:

` cd ext/zlib`  
` ruby extconf.rb --with-zlib-include=/usr/include --with-zlib-lib=/usr/lib`  
` cd ../../`  
` sudo make`  
` sudo make install`

Test this:

` [root@pdb-d rubygems-1.3.7]# ruby --version`  
` ruby 1.8.7 (2009-06-12 patchlevel 174) [i686-linux]`

Step 2: Install [RubyGems](http://rubygems.org/pages/download)
--------------------------------------------------------------

` wget `[`http://production.cf.rubygems.org/rubygems/rubygems-1.3.7.tgz`](http://production.cf.rubygems.org/rubygems/rubygems-1.3.7.tgz)  
` sudo tar xzvf rubygems-1.3.7.tgz`  
` cd rubygems-1.3.7`  
` sudo ruby setup.rb`

Now check with:

` [root@pdb-d rubygems-1.3.7]# gem -v`  
` 1.3.7`

Step 3: Install Rails
---------------------

Note the version! Redmine requires Rails 2.3.5. So:

` gem install rails -v=2.3.5`

Step 4: Install the MySQL gem
-----------------------------

` gem install mysql -- --with-mysql-config=/usr/bin/mysql_config`

Step 5: Install Passenger
-------------------------

Any Rails application can use a bunch of servers.
[Mongrel](http://rubyforge.org/projects/mongrel/) is an example; it runs
completely independent of Apache. [Passenger](http://www.modrails.com/)
runs Rails apps on top of Apache (as a module) is amazing w.r.t. the
simplicity of its setup and debugging.

` gem install passenger`  
` passenger-install-apache2-module`

Now add an `httpd` directive called `/etc/httpd/conf.d/passenger.conf`.
Here are the contents:

` LoadModule passenger_module /usr/local/lib/ruby/gems/1.8/gems/passenger-2.2.14/ext/apache2/mod_passenger.so`  
` PassengerRoot /usr/local/lib/ruby/gems/1.8/gems/passenger-2.2.14`  
` PassengerRuby /usr/local/bin/ruby`

Restart the `httpd` service.

Step 6: Install Redmine
-----------------------

I'm choosing to install Redmine in `/opt`.

`svn checkout `[`http://redmine.rubyforge.org/svn/tags/0.9.3/`](http://redmine.rubyforge.org/svn/tags/0.9.3/)` /opt/redmine`  
`chmod 777 /opt/redmine/files`

-   Configure `/opt/redmine/config/database.yml` to reflect your
    database parameters.
-   Configure `/opt/redmine/config/email.yml` to reflect your
    email parameters.

Tell Passenger to run Redmine!
------------------------------

AFAIK, any Rails app will have a `public` folder that 'kickstarts' the
application (again, don't know anything about this; seems like it to
me.) I created a `/opt/redmine/public/.htaccess` file and added this:

`PassengerAppRoot /opt/redmine`

Now I just symlinked the `/opt/redmine/public` to my Apache root,
restarted `httpd`, and was good to go at <http://localhost/redmine>

`ln -s /opt/redmine/public /var/www/html/redmine`

Sources
-------

-   [Installing Rails on CentOS
    5](http://www.catapult-creative.com/2009/02/04/installing-rails-on-centos-5/)
-   [Deploying a Rails Application with
    Passenger](http://wiki.ocssolutions.com/Deploying_a_Rails_Application_With_Passenger)




