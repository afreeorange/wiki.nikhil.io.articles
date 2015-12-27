All settings are stored in `/etc/3dm2/3dm2.conf`. The default password
is "3ware". It is encrypted as `twOmwmsK8lKk2` like so:

    ROpwd twOmwmsK8lKk2  
    ADMINpwd twOmwmsK8lKk2

Once you set a user and admin password, it changes to another value. To
restore the password to "3ware", first kill all running 3DM2 processes:

    sudo pgrep 3dm2 | xargs kill -9

Then edit the file to restore the default values of `ROpwd` and
`ADMINpwd`. Now start 3DM2

    sudo 3dm2

Bam.
