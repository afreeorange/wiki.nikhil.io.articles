You'll use *your* public key to administer the gitolite server[^1]. I
saved mine in `/tmp/nikhil.pub`.

Installation
------------

    # Create a gitolite service account and switch to it
    useradd git
    su - git

    # Create a folder from where gitolite will run ($HOME/bin should be in $PATH)
    mkdir -p $HOME/bin
    git clone https://github.com/sitaramc/gitolite.git /tmp/gitolite

    # Install the gitolite scripts
    /tmp/gitolite/install -to $HOME/bin

    # Set up gitolite with your key
    gitolite setup -pk /tmp/nikhil.pub

    # That's it! :)

Administration
--------------

    # Clone the admin repository
    git clone git@example.com:gitolite-admin

    # Edit the config file to add/edit repositories and permissions
    vim gitolite-admin/conf/gitolite.conf

    # Add/remove any keys in the key directory
    cp kathy.pub gitolite-admin/keydir/

    # Commit and push. Gitolite will set up remote.
    git add .
    git commit -m "Added blah repo"
    git push

If you see anything weird when cloning the Gitolite admin repository,
hose everything and start over:

    rm -rf ~/.gitolite* ~/bin/* ~/projects.list ~/repositories

Miscellaneous
-------------

If running SSH on a different port, SSH config files are your friend.
Here's a sample `~/.ssh/config`

    Host gitserver
        Hostname example.com
        Port 3574
        User git
        # Don't need if using default id_rsa.pub
        # IdentityFile ~/.ssh/my_key.pub

Then clone with

    git clone gitserver:gitolite-admin
    git clone gitserver:my-repository

Footnotes
---------

<references/>
[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Installation Logs](Category:_Installation_Logs "wikilink")

[^1]: You can simply use `~/.ssh/id_rsa.pub` in your home folder.
    Generate with `ssh-keygen -t rsa`.
