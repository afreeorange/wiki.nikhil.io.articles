When connected to the internet, there exist many ways to look up a word.
[Answers.com](http://answers.com),
[Dictionary.com](http://dictionary.com) are all valid resources if using
a browser. Command line checks (I find) are far faster and there exist
ways to cache a dictionary for really quick lookups if you're on OS X or
Linux (Cygwin on Windows). And it does not stop with dictionaries: you
can add periodic table listings, Wikipedia, the CIA world factbook, the
computer jargon dictionary, WebMD, and many more resources.

### Command-line tricks

On the command line, it is possible to use `curl` to get a dictionary
reference. We can first list all the dictionaries available at
[dict.org](http://dict.org) like so:

` curl `[`dict://dict.org/show:db`](dict://dict.org/show:db)

This lists all available dictionaries and their shortnames:

` gcide "The Collaborative International Dictionary of English v.0.48"`  
` wn "WordNet (r) 2.   `  
` moby-thes "Moby Thesaurus II by Grady Ward, 1.0"`  
` elements "Elements database 20001107"`  
` vera "Virtual Entity of Relevant Acronyms (Version 1.9, June 2002)"`  
` jargon "Jargon File (4.3.1, 29 Jun 2001)"`  
` foldoc "The Free On-line Dictionary of Computing (27 SEP 03)"`  
` easton "Easton's 1897 Bible Dictionary"`  
` hitchcock "Hitchcock's Bible Names Dictionary (late 1800's)"`  
` bouvier "Bouvier's Law Dictionary, Revised 6th Ed (1856)"`  
` devils "THE DEVIL'S DICTIONARY ((C)1911 Released April 15 1993)"`  
` world02 "CIA World Factbook 2002"`  
` .`  
` .`  
` .`  
` eng-fra "English-French Freedict Dictionary"`

To look up a word or phrase from a certain dictionary, issue:

`curl `[`dict://dict.org/d:word_goes_here:dictionary_short_name_here`](dict://dict.org/d:word_goes_here:dictionary_short_name_here)

For instance, I'll attempt to look up India in the CIA World FactBook
(2002 edition):

`curl `[`dict://dict.org/d:India:world02`](dict://dict.org/d:India:world02)` > India.Reference.txt`

And obtain this (partial dump):

` 20 miranda.org dictd 1.9.15/rf on Linux 2.6.26-2-686 `<auth.mime>` <62887065.988.1256582810@miranda.org>`  
` 250 ok`  
` 150 1 definitions retrieved`  
` 151 "India" world02 "CIA World Factbook 2002"`  
` India`  
` `  
` Introduction India`  
` ------------------`  
` Background: The Indus Valley civilization, one `  
` of the oldest in the world, goes`  
` back at least 5,000 years. Aryan`  
` tribes from the northwest invaded`  
` about 1500 B.C.; their merger with`  
` the earlier inhabitants created the `  
` classical Indian culture. Arab`  
` incursions starting in the 8th`  
` century and Turkish in 12th were`  
` followed by European traders`  
` beginning in the late 15th century.`  
` By the 19th century, Britain had `  
` assumed political control of`  
` virtually all Indian lands.`

### Offline Dictionaries

The example above uses online resources. You might, however, need to use
these offline. Like everything else, there's a tool for precisely this
purpose on Linux called [**dict**](http://www.dict.org/bin/Dict) and is
quite flexible in terms of its features. Think client-server
architecture, multiple dictionary references, configurable hosts on the
client side, etc.

The utility is split into two tools: `dictd` (the server daemon) and
`dict` (the client application.)

The `dict` client, by itself, can search the resources at dict.org if
you don't want to install the server. What follows is a guide to
installing the `dictd` daemon and configuring the client app to look at
'itself' (i.e. localhost)

#### Step 1: Obtain and install dict

`  wget `[`ftp://ftp.dict.org/dict/dictd-1.9.15.tar.gz`](ftp://ftp.dict.org/dict/dictd-1.9.15.tar.gz)

Where 1.9.15 is the latest version at the time of this writing. Untar it
and follow the [typical installation
instructions](ftp://ftp.dict.org/dict/INSTALL) to compile this from
source.

**Note**: You might get compiler warnings when running `make` against
lower version numbers [since this is a
bug](http://bugs.gentoo.org/81211).

A few things:

-   If you did not use `--prefix` when running `configure`, the path to
    `dictd` (server) should be `/usr/local/sbin`
-   Configuration files for client and server will be in
    `/usr/local/etc`

#### Step 2: Obtain and set up dictionaries

The dictionary or reference files the `dictd` daemon requires have two
parts: the database and the index files. In this step, we will get the
Webster's 1913 dictionary (which is in public domain).

` wget `[`ftp://ftp.dict.org/dict/pre/dict-web1913-1.4-pre.tar.gz`](ftp://ftp.dict.org/dict/pre/dict-web1913-1.4-pre.tar.gz)

Once you untar the archive, place the two files `web1913.dict.dz` and
`web1913.index` in a convenient location. I placed mine at
`/usr/share/dictionaries`. Make sure these are readable!

` chmod a+r web1913.dict.dz web1913.index`

You can obtain other dictionaries by perusing [the FTP site you got the
Webster's dictionary from](ftp://ftp.dict.org/dict/pre/)

**Note**: `/usr/share/dict` is *not* an appropriate location!
`linux.words` is the file used by the `passwd` command, for instance, to
determine weak passwords.

#### Step 3: Configure dict

You now need to create two configuration files for the dict server and
client: `dictd.conf` and `dict.conf` respectively. These need to go in
`/usr/local/etc`

##### Configure dictd server

`dictd.conf` will contain references to the database(s) from Step 2.

`  database web1913 { data "/usr/share/dictionary/web1913.dict.dz" `  
`                     index "/usr/share/dictionary/web1913.index" }`

If you installed another reference database from [the FTP
site](ftp://ftp.dict.org/dict/pre/) in Step 2, append it to this file
using similar syntax. For instance, if I downloaded a list of chemical
elements, I would add this:

`  database elements { data "/usr/share/dictionary/elements.dict.dz" `  
`                     index "/usr/share/dictionary/elements.index" }`

After this stage, start the dictd server

`  /usr/local/sbin/dictd start`

Test your installation with something

` dictd --test perspicacious`

##### Configure dict client

`dict.conf` can contain a single server or a list of servers which have
the `dictd` daemon running. In this example, we'll use our own machine

` server localhost`

And that's it! Test this with

` dict surreptitious`

### Other notes

You might need an init script if `dict word` does not work.

    DICTD=/usr/local/bin/dictd

    # DICTD_OPTIONS="-put -command_line -options -for -dictd -here"
    DICTD_OPTIONS=""
    DICTD_PID_FILE=/etc/dictd.pid

    case "$1" in
        'start')
            if [ -x $DICTD ]; then
                echo "dictd starting."
                $DICTD $DICTD_OPTIONS
            else
                echo "dictd.init: cannot find $DICTD or it's not executable"
            fi  
        ;;  
        'stop')
            if [ ! -f $DICTD_PID_FILE ]; then
                exit 0
            fi  
            dictdpid=`cat $DICTD_PID_FILE`
            if [ "$dictdpid" -gt 0 ]; then
                echo "Stopping the dictd server."
                kill -15 $dictdpid 2>&1 > /dev/null
            fi  
            rm -f $DICTD_PID_FILE
        ;;  
        *)
            echo "Usage: dictd.init { start | stop }"
        ;;
    esac
    exit 0

### To Do

-   Add information on doing this securely (i.e. dictd should use only
    one port in iptables)



