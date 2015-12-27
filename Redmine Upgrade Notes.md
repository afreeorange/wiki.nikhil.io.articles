Not as peachy as I thought.

## Install log

    Ran gem install ruby -v=2.3.2  
    Complained that rubygems-update was not available.  
     
    Download rubygems-update-1.3.5.gem  
        [http://rubyforge.org/frs/?group_id=126](http://rubyforge.org/frs/?group_id=126)  
        Might need to download previous versions  
        E.g. if at 1.1.0, get 1.2.0, intermittent releases,  
             all the way up tpo 1.3.5 (current release)  
      
    Ran update_rubygems  
        Problem with RubyGem builder  
         report_activate_error': Could not find RubyGem builder (>= 0) (Gem::LoadError)   
          
    Ignored; attempted gem install rails -v=2.3.2  
      
    Was told this required 'rake'  
        [http://rake.rubyforge.org/](http://rake.rubyforge.org/)

    Installed and ran ruby rubygems-update

    Ran gem install rails -v=2.3.2

    Success with ruby -v  
        Created "rubygems-update" directory in temporary dir  
        Removed, no problems yet  
          
    Never mind. Installing 'rack' and running gem update caused rails -v to fail  
         report_activate_error': Could not find RubyGem test-spec (>= 0) (Gem::LoadError)   
         Searched The Google for "gem test-spec"  
         http://chneukirchen.org/blog/archive/2007/01/announcing-test-spec-0-3-a-bdd-interface-for-test-unit.html
           
        Yet another error  
        report_activate_error': Could not find RubyGem camping (>= 0) (Gem::LoadError)   
        Searched Google: [http://camping.rubyforge.org/files/README.html](http://camping.rubyforge.org/files/README.html)  
          
        Removed repository from .gemrc in /root: http;//gems.rubyforge.org  
        Was stalling  
        Put it back in with gem source -a [http://gems.rubyforge.org/](http://gems.rubyforge.org/)  
          
        Running gem install camping worked  
          
    extconf.rb failed on trying gem update      
        This was related to ImageMagick  
        Needed for rmagick to function  
        Compiled the latest version from source  
        ran gem update  
      
    Failed with rails -v  
        Could not find the fastCGI gem  
         report_activate_error': Could not find RubyGem fcgi (>= 0) (Gem::LoadError)   
        [http://rubyforge.org/frs/?group_id=926&release_id=5851](http://rubyforge.org/frs/?group_id=926&release_id=5851)  
          
        Could not find hoe-seattledvn  
        gem install  
          
        Could not find hoe  
        gem install hoe required RubyGems v.1.3.0 and above  
        [http://rubyforge.org/frs/?group_id=126&release_id=37073](http://rubyforge.org/frs/?group_id=126&release_id=37073)  
        Untar, run ruby setup.rb  
          
    Ran update_rubygems  
    Ran rails -v  
    Success: gem at v.1.3.5, Rails at v.2.3.4, Ruby at v.1.8.5

## Upgrade Redmine

    RAILS_ENV=production rake db:migrate_plugins  
        NEEDED Rails 2.1.2  
        gem install rails -v=2.1.2

-   MAKE SURE that all files are owned by mongrel:mongrel

        chown -R mongrel:mongrel your_rails_app

-   Mongrel app server was originally started with:

        /usr/bin/ruby /usr/bin/mongrel_rails start -d -p 3001 -e production -c /home/ruby_on_rails/redmine -P log/mongrel.pid --user mongrel --group mongrel --prefix=/redmine

-   Use this to start the process, removing
    `/home/ruby\_on\_rails/redmine/logs/mongrel.pid` if you run into
    any errors about this file.

Change to the **/home/rub\_on\_rails** directory before issuing the
command.

    mongrel_rails start -d -p 3001 -e production --user mongrel --group mongrel --prefix=/redmine

## Other Notes

-   Sometimes, deleting `source_cache` helps with update process
-   Location ascertained by gem env. Default at `/usr/<local>/lib/ruby`
-   [Might also need to install 'rack' to run gem
    update](http://rubyforge.org/frs/?group_id=3113&release_id=40490)

## Fixing the "Mongrel has to be restarted" issue

    Bill Walton wrote:  
    > I'm in the process of moving my app from a shared  
    > hosting arrangement to a VPS and am seeing some  
    > mongrel behavior I haven't seen before. Specifically,  
    > after about 30 minutes of inactivity mongrel goes  
    > unresponsive and has to be restarted.  
    Zed A. Shaw wrote:  
    > make sure you've got the compiled mysql gem and  
    > you aren't using the rails mysql.rb file.  
    Ezra Zygmuntowicz wrote:  
    > Also if your host is running linux then you can tell for  
    > sure if it's the compiled driver from irb:  
    >  
    > ey00-s00059 ~ # irb  
    > irb(main):001:0> require 'mysql.so'  
    > => true  
    Bill Walton wrote:  
    > When I run 'gem list' the list includes:  
    > mysql (2.7)  
    >  
    > BUT ...  
    >  
    > [EMAIL PROTECTED] [~/emrec]# irb  
    > irb(main):001:0> require 'mysql.so'  
    > LoadError: no such file to load -- mysql.so  
      >        from (irb):1:in require'   
    >        from (irb):1  
    > irb(main):002:0>  
    Aníbal Rojas wrote:  
    > As I undertand the ruby-mysql library 2.7 does  
    > require the libmysqlclient to be properly installed,  
    > and depending on the location of mysql.so it is  
    > possible that rubygems is unable to load it, while  
    > the library is actually using it...  
    Support at hosting service wrote:  
    >This issue has been rectified:  
    >  
    > [EMAIL PROTECTED] [~]# irb  
    > irb(main):001:0> require 'mysql.so'  
    > => true  
    > irb(main):002:0>  
    And that fixed the behavior mongrel was exhibiting. 

* http://www.redmine.org/boards/2/topics/9292
* http://www.redmine.org/issues/4144

### Resources

* http://www.catapult-creative.com/2009/02/04/installing-rails-on-centos-5/
* http://discuss.joyent.com/viewtopic.php?id=23748
* http://www.elctech.com/articles/-solved-rightscale-centos-5-0-rails-2-3-4-upgrade
* http://wiki.opscode.com/display/chef/Installation+on+CentOS+5.2+with+gems+%28In+progress%29
* http://forum.slicehost.com/comments.php?DiscussionID=672
* http://forum.slicehost.com/comments.php?DiscussionID=672
* http://www.centos.org/modules/newbb/viewtopic.php?topic_id=11821
