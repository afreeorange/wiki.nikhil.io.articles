-   The flowchart's really helpful in visualizing how Varnish handles a
    request before asking the backend.
-   `default.vcl` comes with commented out subroutines which are worth
    a study.
    -   I commented the hell out of them in this repo.
-   Test code with

`   varnishd -C -f /etc/varnish/default.vcl`

Pre-Flight
----------

`   # Install the repo`  
`   rpm -ivh `[`http://repo.varnish-cache.org/redhat/varnish-3.0/el5/noarch/varnish-release/varnish-release-3.0-1.noarch.rpm`](http://repo.varnish-cache.org/redhat/varnish-3.0/el5/noarch/varnish-release/varnish-release-3.0-1.noarch.rpm)  
`   `  
`   # Install Varnish`  
`   yum -y install varnish`  
`   `  
`   # Make copy of config`  
`   cp /etc/varnish/default.vcl{,.original}`

Can now start a varnish daemon with `varnishd` and specify the start
params [like the documentation asks you to
do](https://www.varnish-cache.org/docs/3.0/tutorial/starting_varnish.html).
But will use init daemons and sysconfig options on a RHEL box, which is
neater.

Edit `/etc/sysconfig/varnish`. Changed some default options:

`   # Listen on all addresses, on the HTTP port`  
`   VARNISH_LISTEN_ADDRESS=`  
`   VARNISH_LISTEN_PORT=80`  
` `  
`   # Use a 250MB cache`  
`   VARNISH_STORAGE_SIZE=250M`  
`        `  
`   # Use malloc`  
`   VARNISH_STORAGE="malloc,${VARNISH_STORAGE_SIZE}"`

Both "file" and "malloc" use disk and memory, [but in different
ways](https://www.varnish-software.com/static/book/Tuning.html#storage-backends).

This is the architecture:

`   World ---> Varnish ---> Nginx ---> Application Server`  
`              0.0.0.0:80`  
`                           127.0.0.1:8080`  
`                                      127.0.0.1:9090`

In this case, the Nginx server is considered a "backend server" from
which Varnish can request and then get data. I defined a simple one in
`/etc/varnish/default.vcl`

    # Define a simple Nginx backend
    backend nginx_server {
            .host = "localhost";
            .port = "8080";
    }

Restarted the varnish and nginx services, made sure they were listening
on the right ports.

`   [root@nikhil conf.d]# netstat -tunlp | grep 80`  
`   tcp  0   0 0.0.0.0:80        0.0.0.0:*     LISTEN   27090/varnishd`  
`   tcp  0   0 127.0.0.1:8080    0.0.0.0:*     LISTEN   26861/nginx`

The site should now work exactly as before. A few notes:

-   There's no caching taking place. Yet. That's where the VCL
    [DSL](http://en.wikipedia.org/wiki/Domain-specific_language)
    comes in.
-   Since I only have one server, I'm not interested in Varnish's
    load-balancing capabilities.

Methods
-------

### vcl\_recv

The entry point. I played around with the default subroutine by simply
adding this to `default.vcl` and restarting Varnish:

    sub vcl_recv {
        # Check for a standard HTTP verb. If none used, bark.
        if (req.request != "GET") {
                error 400 "I don't understand what you want me to do.";
        }
    }

After that, a simple `curl -X POST nikhil.io` yielded:

        <?xml version="1.0" encoding="utf-8"?>
        <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
         "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
        <html>
          <head>
            '''<title>400 I don't understand what you want me to do.</title>'''
          </head>
          <body>
            <h1>Error 400 I don't understand what you want me to do.</h1>
            <p>I don't understand what you want me to do.</p>
            <h3>Guru Meditation:</h3>
            <p>XID: 2033234310</p>
            <hr>
            <p>Varnish cache server</p>
          </body>
        </html>

Nice.

### vcl\_hash

Varnish uses a key-value memory map. This subroutine defines how the key
is generated. By default, it uses (URL or IP) + hostname.

### vcl\_pipe and vcl\_pass

Called when `return(pipe)` or `return(pass)` are... returned from
`vcl_recv`.

In pipe mode is the simplest "Varnish-bypass" mode, where it
short-circuits the connection between the client and the backend. No
caching, no logging. Varnish does nothing while client speaks to backend
(still through Varnish.)

In pass mode, Varnish can look at (and manipulate) request and response
data

### vcl\_fetch

-   A full restart wipes the cache: neither "file" nor "malloc"
    are persistent.
-   You can see how Varnish translates the VCL into C code by:

`   varnishd -C -f /etc/varnish/default.vcl`

-   Here's a useful [list of
    actions](https://www.varnish-cache.org/docs/3.0/tutorial/vcl.html#actions).
