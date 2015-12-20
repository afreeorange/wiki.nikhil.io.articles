Extremely useful in scripting. For a full listing of signals, use
`kill -l`. Here's a sample script:

    #!/bin/bash
    # Sample script to test the trap command

    # Catch and report errors
    error_keyboard_interrupt() {
          echo -e "Caught SIGINT; logging and exiting"
          TEMPLOG=$(mktemp XXXXXX)

          echo -e "$1 Caught keyboard interrupt!" > $TEMPLOG
          mail -s "Severe" user@eng.uiowa.edu < $TEMPLOG

          rm -f $TEMPLOG
          exit
    }

    TESTPARAM="Oh noes!"
    trap $(error_keyboard_interrupt $TESTPARAM) SIGINT

    while :  # Same as while true
    do
          sleep 60
          echo -e "Still alive..."
    done

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
