Miscellaneous
-------------

### Custom CA Bundle with `requests`

Set this environment variable and request away as usual:

    export REQUESTS_CA_BUNDLE="/path/to/ca-bundle.pem"

Tricks
------

### Check dictionary for multiple keys

    if not all([key in dictionary for key in ['key1','key2','key3']]):
        raise Exception('Missing keys, yo')

### Test SMTP Server

    sudo python -m smtpd -n -c DebuggingServer localhost:25

### Singular & Plural

    # Simple
    print "This took %d day%s" % (days, "s"[days==1:])

    # Tough
    n = 1
    print "I see %d %s" % (n, ['octopus', 'octopuses'][n!=1])

### Path to `site-packages`

    import site; site.getsitepackages()
    ['/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages']
