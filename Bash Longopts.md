    #!/bin/bash

    script_usage() {
    cat <<EOF
    USAGE `basename $0` [ OPTIONS ]
       -f1 --flag1
            Sets a parameter
       -f2 --flag2 PARAM
            Provides a parameter
    EOF
    }

    if [ -z "$1" ]; then
        script_usage
    else
        while [ -n "$1" ]; do
            case "$1" in
                
                --flag1|-f1)
                    echo -e "Flag1 set"
                ;;

                --flag2|-f2)
                    shift
                    flag2_set=1
                    if [ -z "$1" ]; then
                        echo -e "Specify an argument"
                    else
                        echo -e "Flag2 specified to be $1"
                    fi
                ;;
                
            esac
            shift
        done
    fi

[Category:Nikhil's Notes](Category:Nikhil's_Notes "wikilink")
[Category:From a past sysadmin
life](Category:From_a_past_sysadmin_life "wikilink")
