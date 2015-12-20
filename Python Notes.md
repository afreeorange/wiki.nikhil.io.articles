Miscellaneous
-------------

### Custom CA Bundle with `requests`

Set this environment variable and request away as usual:

    export REQUESTS_CA_BUNDLE="/path/to/ca-bundle.pem"

Tricks
------

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

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Python](Category:_Python "wikilink")
