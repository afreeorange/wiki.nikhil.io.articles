In this guide, I'll be installing `mod_pagespeed` on a 64-bit CentOS 5.5
system. The host is `example.com`.

Installation
------------

Grab [the most appropriate RPM](http://code.google.com/speed/page-speed/download.html)
and install it.

    rpm -ivh https://dl-ssl.google.com/dl/linux/direct/mod-pagespeed-beta_current_x86_64.rpm

This is the RPM manifest on a 64-bit system:

    /etc/cron.daily/mod-pagespeed  
    /etc/httpd/conf.d/pagespeed.conf  
    /usr/lib64/httpd/modules/mod_pagespeed.so  
    /var/www/mod_pagespeed/cache  
    /var/www/mod_pagespeed/files

Testing
-------

To check for a proper install, you can do two things:

*   Check if the `/var/www/mod_pagespeed` directories are populated, or
*   Use `curl` or `wget` to check for the appropriate header.

### Checking headers

Let's use `wget`:

    wget -O - --server-response http://example.com/home/index.php > /dev/null 

Here's the response:

    --2011-01-05 09:01:36--  http://example.com/home/index.php
    Resolving example.com... 128.255.22.132
    Connecting to example.com|128.255.22.132|:80... connected.
    HTTP request sent, awaiting response... 
      HTTP/1.1 200 OK
      Date: Wed, 05 Jan 2011 15:01:36 GMT
      Server: Apache/2.2.3 (CentOS)
      X-Powered-By: PHP/5.2.10
      X-Mod-Pagespeed: 0.9.11.5-293
      Vary: Accept-Encoding
      Content-Length: 4657
      Connection: close
      Content-Type: text/html; charset=UTF-8
    Length: 4657 (4.5K) [text/html]
    Saving to: `STDOUT'
    
    100%[========================================>] 4,657       --.-K/s   in 0s      
    
    2011-01-05 09:01:36 (211 MB/s) - `-' saved [4657/4657]

The `X-Mod-Pagespeed` header should tell you that pagespeed is in
action.

Tweaking Pagespeed
------------------

Pagespeed has 18 'filters' with which you can tweak for performance. For
example, I can remove all HTML comments with this filter in
`/etc/httpd/conf.d/pagespeed.conf`

    ModPagespeedEnableFilters remove_comments

To see a "before-and-after", append `?ModPagespeed=off` to any page
served up. [This page](http://www.modpagespeed.com/) does a good job of
explaining other filters. You can also check the documentation.

### Viewing statistics

The `/etc/httpd/conf.d/pagespeed.conf` config defines
`/mod_pagespeed_statistics` as a page where you can take a look at
pagspeed's statistics.

Sources
-------

*   [Installing mod\_pagespeed on CentOS (cPanel/WHM)](https://fusi0n.org/linux/installing-googles-mod_pagespeed-on-centos-with-cpanel-and-whm)
