Pre-Flight
----------

-   Installation was on a 32-bit CentOS 5.6 box.
-   I'm assuming that you have a MySQL database already set up.
-   The Snorby Root is `/opt/snorby`

Compiling ImageMagick
---------------------

Snorby requires a version that the yum repos do not (predictably) have.
Your best option is to compile from the source ([or from an
SRPM](http://www.imagemagick.org/Usage/api/#building)
[\[2](http://en.citizendium.org/wiki/User:Dan_Nessett/Technical/Upgrade_to_1.16#ImageMagick_6.6.2-10)\]):

    yum -y groupinstall "Development Tools"
    wget -O - ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz | tar -xzvf -
    cd ImageMagick-6.6.9
    ./configure
    make
    make install

Installation Log
----------------

    # Install Git & Development Libs
    yum -y install git gcc gcc-c++ curl-devel httpd-devel \
                   apr-devel apr-util-devel zlib-devel \
                   libxml2 libxml2-devel libxslt \
                   libxslt-devel expat-devel gettext-devel \
                   mysql-devel readline-devel

    # Compile and Install Ruby
    wget ftp://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.2-p0.tar.gz
    tar -xvzf ruby-1.9.2-p0.tar.gz 
    cd ruby-1.9.2-p0 
    ./configure 
    make && make install
    cd ..
    rm -rf ruby-1.9.2-p0*

    # Get Snorby 2
    cd /opt
    git clone git://github.com/Snorby/snorby.git
    cd snorby/
    git pull

    # Some required gems
    gem install rails mysql

    # Install the Gem Bundler (yum for Ruby)
    gem install bundler
    cd /opt/snorby
    bundle install

    # Symlink to Apache site root
    ln -s /opt/snorby/public /var/www/html/snorby

    # Install Passenger (Ruby Server)
    gem install passenger –-no-rdoc –-no-ri

    # PDF export support
    wget http://wkhtmltopdf.googlecode.com/files/wkhtmltopdf-0.9.9-static-amd64.tar.bz2
    tar -xjvf wkhtmltopdf-0.9.9-static-amd64.tar.bz2
    mv wkhtmltopdf-amd64 /usr/local/bin/wkhtmltopdf
    # Now edit /opt/snorby/config/snorby_config.yml to match the location of this file

### Preparing Apache

` `**`#` `Start` `the` `extremely` `friendly` `Passenger` `Apache`
`Module` `Installer`**  
` passenger-install-apache2-module`

Now add this to `/etc/httpd/conf.d/passenger.conf`:

` LoadModule passenger_module /usr/local/lib/ruby/gems/1.9.1/gems/passenger-3.0.7/ext/apache2/mod_passenger.so`  
` PassengerRoot /usr/local/lib/ruby/gems/1.9.1/gems/passenger-3.0.7`  
` PassengerRuby /usr/local/bin/ruby`

Now add this to `/opt/snorby/public/.htaccess`:

`   RailsBaseURI /snorby`  
`   PassengerAppRoot /opt/snorby`

### Preparing MySQL

Edit `/opt/snorby/config/database.yml` to match your MySQL connection
params.

-   **Maintaining spacing is bloody important** and will save you a lot
    of grief.
-   **Use the `root` account** to your MySQL Instance for the
    *initial* setup.

### Initialize Snorby

Go to `/opt/snorby` and set up the databases:

` rake snorby:setup RAILS_ENV=production`

### Reconfiguring MySQL

You can now remove `root` from the database config. Create a user (I
called mine "snorby") and a password with full privileges to the
"snorby" database.

Errors
------

### EZPrint error

` "`[`http://github.com/mephux/ezprint.git`](http://github.com/mephux/ezprint.git)` (at rails3) is not checked out. `  
``  Please run `bundle install` (Bundler::GitError)" ``

This [has been documented](https://github.com/Snorby/snorby/issues/6).
Go to the Snorby root and do this:

` cd /opt/snorby`  
` bundle pack`  
` bundle install --path vender/cache`

Take note that it's vend**e**r and not vend**o**r. I'm guessing this was
a developer typo.

### undefined method \`\[\]' for nil:NilClass

` [root@example snorby]# rake snorby:setup RAILS_ENV=production`  
` (in /opt/snorby)`  
` rake aborted!`  
``  undefined method `[]' for nil:NilClass ``

This went away after I actually indented `config/database.yml` properly.

### Worker doesn't start via Web Interface

-   Make sure you have Rails installed (`gem install rails`)
-   Make sure that `readline` compiled and installed with Ruby
    -   I had errors like
        `` `require': no such file to load -- readline `` when trying to
        start the Rails console

` # Install the correct yum package`  
` yum -y install readline-devel`  
` `  
` # Download, extract Ruby and head into the readline folder`  
` wget `[`ftp://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.2-p0.tar.gz`](ftp://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.2-p0.tar.gz)  
` tar -xvzf ruby-1.9.2-p0.tar.gz `  
` cd ruby-1.9.2-p0/ext/readline`  
` `  
` # Now compile and install it`  
` ruby extconf.rb`  
` make`  
` make install`

When you go to `/opt/snorby` and pull up the Rails console, you
shouldn't have any errors:

` [root@example snorby]# rails c`  
` Loading development environment (Rails 3.0.5)`  
` irb(main):001:0>`

You can manually start the worker using these commands:

` Snorby::Worker.stop      # Stop The Snorby Worker`  
` Snorby::Worker.start     # Start The Snorby Worker`  
` Snorby::Worker.restart   # Restart The Snorby Worker`

If you have Rails installed, the application should automagically start
the worker threads.




