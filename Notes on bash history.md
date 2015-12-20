There are three totally wonderful things you can do to enhance the
utility of your `.bash_history` file. Add this block to your `.bashrc`:

<bash>

` # Keep 10,000 commands worth of history`  
` export HISTSIZE=10000`  
` `  
` # Make 10,000 lines more awesome by erasing duplicate commands`  
` export HISTCONTROL=erasedups`  
` `  
` # Don't lose command history from across many sessions`  
` shopt -s histappend`

</bash>

By itself, the last option appends that particular bash process' history
to `.bash_history` at the end of the session. But combined with the
`HISTCONTROL` option, it's more like an intelligent 'merge' than a
senseless append. Neato! Recursive searches with `Ctrl + r` will be
*much* better from now.

### Other stuff you could do

To see your history file, you can either `vi` it like a caveman, or be a
hip 80s dude by issuing this:

<bash>

` history`

</bash>

Here's sample (truncated) output:

<bash>

` 873  cat smbd.log`  
` 874  service smb status`  
` 875  netstat -tulpn`  
` 884  find /media/pool02/asap -type d | sort | grep '\<[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}T[0-9]\{2\}\.[0-9]\{2\}\.[0-9]\{2\}\>'`  
` 891  eval ssh tigris "df -h" | grep -c / | sed "s/./ &/g"`  
` 902  date`

</bash>

Now I can just run that long `find` command by merely issuing:

<bash>

` !884`

</bash>

You can actually merge histories across two or more 'live' sessions by
hand:

<bash>

` history -a; history -n`

</bash>

I've read of people having this happen automagically by setting the
`PROMPT_COMMAND` variable to the snippet above. Setting this variable
executes its value before each new prompt.

<bash>

` PROMPT_COMMAND="history -a; history -n"`

</bash>

-   `history -a` appends local history changes to `.bash_history`
-   `history -n` fetches changes from `.bash_history`

This has not worked for me on CentOS; YMMV.

### Clearing History

<bash>

` history -c && rm -f ~/.bash_history`

</bash>

This is because `bash` stores history in memory and in a file.

### History with Timestamps

<bash> export HISTTIMEFORMAT="%F %T " </bash>

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
