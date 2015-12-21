Tried this with a [Flask](http://flask.pocoo.org/) app.

`   location /my_app {`  
`       proxy_pass `[`http://127.0.0.1:5000/`](http://127.0.0.1:5000/)`;`  
`       proxy_set_header  X-Real-IP  $remote_addr;`  
`   }`

Key idea is the trailing slash.


