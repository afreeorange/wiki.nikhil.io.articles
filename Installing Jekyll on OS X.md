-   Install Xcode
-   Run Xcode, go to **Preferences**
    -   Sign-in under "Accounts"
    -   Tab over to "Downloads" and click the teeny arrow for "Command
        Line Tools"
-   Once installed, install Homebrew

` ruby -e "$(curl -fsSkL raw.github.com/mxcl/homebrew/go)"`

-   Then install the Ruby Version Manager[^1]

` curl -L `[`https://get.rvm.io`](https://get.rvm.io)` | bash -s stable --ruby`

-   Source the RVM:

` source /usr/local/rvm/scripts/rvm`

-   Install Jekyll

` gem install jekyll`

-   Test. I did it by running `jekyll serve` from a
    [Bootstrap](http://getbootstrap.com/) download[^2].

### Footnotes

<references />



[^1]: I temporarily had to `chown root:admin /usr/local/bin/brew` for
    this to work (and then changed it back)

[^2]: <http://localhost:9001>
