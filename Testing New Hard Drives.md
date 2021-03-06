Find out the disks on the system

    cat /var/run/dmesg.boot

Run the short test in the background[^1].

    smartctl -t short /dev/ada0

Then run a conveyance test (for any damage during shipping)

    smartctl -t conveyance /dev/ada0

Then check for [bad
blocks](https://wiki.archlinux.org/index.php/Badblocks)[^2][^3].

    badblocks -ws /dev/ada0

Then run a long test[^4].

    smartctl -t long /dev/ada0

The `-a` flag shows you everything about the drives

    smartctl -a /dev/ada0

including the time remaining for the tests

    Self-test execution status:      ( 249) Self-test routine in progress...  
                                            90% of test remaining.

and the time it would take to run the tests:

    Short self-test routine  
    recommended polling time:    (   2) minutes.  
    Extended self-test routine  
    recommended polling time:    ( 529) minutes.  
    Conveyance self-test routine  
    recommended polling time:    (   5) minutes.

To see the results *after* the tests have run

    smartctl -l selftest /dev/ada0

References
----------

*   [Interpreting `badblocks` data](https://forums.freenas.org/index.php?threads/interpreting-badblocks-output.27421/).

Footnotes
---------

[^1]: Use `-C` to run in foreground. But then again, why would you?

[^2]: The non-destructive version is `badblocks -ns`.

[^3]: This is a 2-phase, 4-pass command that will take a *long* time. On
    a 4TB WD Red, the whole process was done in about 75 hours.

[^4]: This took about 8 hours on a 4TB WD Red.
