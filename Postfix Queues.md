### Listing

` postqueue -p`  
` `  
` # This will work as well`  
` mailq`

### Deleting All Messages

` postsuper -d ALL`

### Deleting all from Specific Address

` mailq | tail +2 | grep -v '^ *(' | awk  'BEGIN { RS = "" } { if ($8 == "`**`email@address.com`**`" && $9 == "") print $1 } ' | tr -d '*!' | postsuper -d -`



