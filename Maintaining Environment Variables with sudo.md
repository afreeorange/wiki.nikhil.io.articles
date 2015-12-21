Especially useful if you need to use proxies. E.g., this will fail since
the `sudo` user doesn't know about the `https_proxy` variable.

` sudo curl -L `[`https://get.rvm.io`](https://get.rvm.io)` | bash -s stable --ruby`

The solution is to `visudo`, find this

` Defaults env_reset `

and add this for each variable you want available.

` Defaults env_keep += "variable1 variable2"`

In the case of proxies,

` Defaults env_keep += "http_proxy https_proxy ftp_proxy"`



