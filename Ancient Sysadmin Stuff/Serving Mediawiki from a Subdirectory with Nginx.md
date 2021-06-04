-   Mediawiki is installed in `/var/www/html/mediawiki`
-   URL is `example.com/wiki`

### `/etc/nginx/conf.d/wiki.conf`

    server {
        listen 80;
        server_name example.com;

        root   /alternate/var/www/html;
        index  index.html index.htm index.php;

        # Redirect to Mediawiki install path
        location /wiki {
            try_files $uri $uri/ @wiki_rewrite;
        }
        location @wiki_rewrite {
            rewrite ^/wiki/(.*)$ /mediawiki/index.php?title=$1&$args;
        }
    }

### `LocalSettings.php`

        $wgServer      = "http://example.com";
        $wgScriptPath  = "/mediawiki";
        $wgArticlePath = "/wiki/$1";
        $wgUsePathInfo = true;
