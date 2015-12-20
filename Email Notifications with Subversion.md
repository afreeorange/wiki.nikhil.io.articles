Pre-Flight
----------

-   Running SubVersion 1.4.2 on CentOS 5.5 (x64)
-   For this guide, my repository is located at `/home/svn/repository`
-   I'll be using `mailer.py` instead of `commit-email.pl`
    -   The former is more configurable and can be used on a
        per-directory basis. This is useful if you have many projects in
        a single repo.

Setup
-----

### Copy over the sample `post-commit` hook

**Permissions and ownership** are vitally important here!

` cp hooks/post-commit.tmpl hooks/post-commit`  
` chmod 770 hooks/post-commit`  
` chown apache:apache hooks/post-commit`

Now edit `hooks/post-commit` and add this line:

` /usr/share/doc/subversion-1.4.2/tools/hook-scripts/mailer/mailer.py commit "$REPOS" "$REV"`

Comment out these lines:

` commit-email.pl "$REPOS" "$REV" commit-watchers@example.org`  
` log-commit.py --repository "$REPOS" --revision "$REV"`

### Configure email alerts for a repository

First copy over the sample mailer config:

` cp /usr/share/doc/subversion-1.4.2/tools/hook-scripts/mailer/mailer.conf.example conf/mailer.conf`

A general file looks like this:

` [general]`  
` mail_command = /usr/sbin/sendmail`  
` smtp_hostname = localhost`  
` `  
` [defaults]`  
` diff = /usr/bin/diff -u -L %(label_from)s -L %(label_to)s %(from)s %(to)s`  
` commit_subject_prefix = "[SVN - IT Scripts]"`  
` propchange_subject_prefix =`  
` lock_subject_prefix =`  
` unlock_subject_prefix =`  
` from_addr = donotreply@svn.eng.uiowa.edu`  
` to_addr = nikhil-anand@uiowa.edu anand.nikhil@gmail.com`  
` reply_to = donotreply@svn.eng.uiowa.edu`  
` generate_diffs = add copy modify`  
` show_nonmatching_paths = yes`  
` suppress_deletes = yes`  
`   `  
` [maps]`

The various options should be self-explanatory. A test commit should
work at this point.

### Configuring alerts for specific folders

Let's say that a specific group of people is to know if commits are made
to the `firewall_scripts` folder. In that case:

` [firewall_scripts]`  
` for_paths = ^firewall_scripts($|/)`  
` commit_subject_prefix = "[SVN - Firewall Scripts]"`  
` from_addr = donotreply@svn.eng.uiowa.edu`  
` to_addr = it-admins@genome.uiowa.edu`

### Testing things out

You don't necessarily need to commit things to test out the config.

` sh hooks/post-commit /home/svn/repository 195`

Where the last two arguments are the path to the repo and the revision
number.

Tip of the Iceberg
------------------

There are *tons* of things you can do with `mailer.conf` in a large
and/or complicated setting. The best way to get acquainted with its
features is to essentially read the default file to figure out what it's
capable of. Reading others' config files also helps :)

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:Installation Logs](Category:Installation_Logs "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
