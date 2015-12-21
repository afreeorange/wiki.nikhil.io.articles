MinorDomo is a "minimalistic replacement for
[MajorDomo](http://www.greatcircle.com/majordomo/)".

In this guide:

-   I have my MinorDomo lists at `/home/minordomo`
-   I'll be creating a list called `developers@example.com`
    -   All messages sent to this list will be archived
    -   Only individuals on the list can send messages to the list (i.e.
        it's not 'open')

Creating a new mailing list
---------------------------

First copy over the sample list to the MinorDomo lists folder:

` cd /home/minordomo`  
` cp -r /usr/doc/minordomo-0.6.1/sample-list developers`

Here's a table of entities in the folder you just copied. **Keep in
mind** that some of these might not exist!

| Name | Type | Function |-------- | `list` | File | Contains email addresses, one per line |-------- | `footer` | File | Contains text which is to be appended to every message sent to this list (e.g. disclaimer, unsubscribe info). Can use placeholders for administrator email (`\a`), domain name (`\d`), mailing list name (`\l`, sans domain), lisr URI (`\u`). |-------- | `info` | File | Information on the list. Obtained by an email to the list with subject line "**info developers**" |-------- | `one-liner` | File | A one-line description of the list |-------- | `config` | File | List configuration file. If present, overrides `/etc/minordomo.conf` Two important directives: 
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        -   `open_lists=none` makes this a 'closed' list                                                
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        -   `subject_prefix=[Developers List]` gives each message a prefix "Developers List"            
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |--------                                                                                       | `archive` | Directory | All messages sent to the list are saved in this folder. It is not created when you copy the sample list above. Messages are stored in YYYY/MM/DD/ sub-folders. You can get awesome HTML output with [HyperMail](http://www.hypermail-project.org/hypermail.html) |
|------|------|--------------------|--------|------|--------------------------------------------------|----------|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|------|-------------------------------------------------------------------------------------------------------------|-------------|------|----------------------------------------------|----------|------|------------------------------------------------------------------------------------------------|-----------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|

-   Once you're done editing things to your heart's content, **change
    the permissions** to `nobody:nobody`.
-   Then **add the following** to `/etc/aliases`:

` developers: "|/usr/sbin/minordomo.pl developers"`

-   Rebuild the aliases DB by **issuing** `newaliases`

Now test the list with an email.

Working with MinorDomo Lists
----------------------------

Sending an email to `minordomo@example.com` with the following subject
lines does a variety of things:

| Subject Line | Action |-------- | subscribe <list> | subscribes to a mailing list |-------- | unsubscribe <list> | unsubscribes from a mailing list |-------- | info \[<list>\] | gets information on a list or server |-------- | list <list> | returns a subscriber list if enabled |
|--------------|------------------|------------------|----------------------------------------|--------------------|--------------------------------------------|-----------------|------------------------------------------------|-------------|--------------------------------------|

Sample HyperMail Script
-----------------------

    #!/usr/bin/perl -w

    umask 0000;

    $archive_dir = "/home/minordomo/archive";
    $output_dir = "/home/minordomo/archive/html";

    @years = `ls $archive_dir`;
    chomp @years;

    foreach $year (@years) {

            unless (-e "$output_dir/$year") {
                    mkdir "$output_dir/$year";
            }
            `find $archive_dir/$year -type f | xargs cat | /usr/local/bin/hypermail -x -d $output_dir/$year`;
    }



