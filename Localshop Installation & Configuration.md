Localshop Installation & Configuration
======================================

You will need Python 2.7.11 for [localshop](http://localshop.readthedocs.io/).

Installation
------------

    pip install localshop

Now [update `kombu`](https://github.com/mvantellingen/localshop/issues/172) to prevent errors when you initialize localshop.

### Start Celery

    localshop celery worker -B -E

### Initialize localshop

By default, the localshop database, packages, and config are kept in `~/.localshop`. To change this,

    export LOCALSHOP_HOME=/path/to/new/root

Then initialize with

    localshop init

### Start localshop

    gunicorn localshop.wsgi:application

Configuration
-------------

To give access to everyone, at least initially, set a CIDR of `0.0.0.0/0`

### Publishing Packages

Under Permissions -> Credentials, set yourself up with a keypair. Then [create a `~/.pypirc`](http://localshop.readthedocs.io/en/latest/how_it_works.html) that looks something like

```
[distutils]
index-servers =
    pypi-local

[pypi-local]
repository: http://localhost:8000/simple/
username: <access_key>
password: <secret_key>
```

### Installing packages

Add this to `~/.pip/pip.conf`

```
[global]
index-url = http://<access_key>:<secret_key>@localhost:8000/simple
```
