Find out the disks on the system

` cat /var/run/dmesg.boot`

Run the short test in the background[^1].

` smartctl -t short /dev/ada0`

Then run a conveyance test (for any damage during shipping)

` smartctl -t conveyance /dev/ada0`

Then check for [bad
blocks](https://wiki.archlinux.org/index.php/Badblocks)[^2]

` badblocks -ws /dev/ada0`

Then run a long test

` smartctl -t long /dev/ada0`

The `-a` flag shows you everything about the drives

` smartctl -a /dev/ada0`

including the time remaining for the tests

` Self-test execution status:      ( 249) Self-test routine in progress...`  
`                                         90% of test remaining.`

and the time it would take to run the tests:

` Short self-test routine`  
` recommended polling time:    (   2) minutes.`  
` Extended self-test routine`  
` recommended polling time:    ( 529) minutes.`  
` Conveyance self-test routine`  
` recommended polling time:    (   5) minutes.`

To see the results *after* the tests have run

` smartctl -l selftest /dev/ada0`

Footnotes
---------

<references markdown="1">
[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: FreeNAS](Category:_FreeNAS "wikilink")

[^1]: Use `-C` to run in foreground. But then again, why would you?

[^2]: The non-destructive version is `badblocks -ns`.
