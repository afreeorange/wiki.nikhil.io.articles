Introduction
------------

In this guide, I will set up secure, keyless SSH access from a test
server, **server.example.com** to a client **client.example.com**.

I type "secure" since ordinary keyless SSH setups do *not* involve
entering a password for the SSH key. It is easy to see how this is
useful for `cron` jobs, for example, which might need access to client
machines. This also means that if the server is compromised, it is
trivially easy to gain access to its keyless clients.

This situation can be improved greatly with strong passwords,
`ssh-agent` and its delightful front-end,
[`keychain`](http://www.funtoo.org/Keychain).

Pre-flight - Pertinent folders and permissions
----------------------------------------------

Explained later

### Commands

*   `ssh-keygen` - Used to generate keys
*   `ssh-copy-id` - Copy a key from the server to client (CentOS *only*)

### Files and Folders

*   `<User Home Folder>/.ssh` - Folder which stores all keys for the
    user
    *   The access permissions *need* to be `700`!
*   `<User Home Folder>/.ssh/authorized_keys` - Plain text file with
    public keys authorized to ssh into the machine as this user
    *   The access permissions *need* to be `600`!

### This is Ridiculously *Crucial*

*   It is *vitally* important that the permissions on the `.ssh` folder
    and `authorized_keys` are `700` and `600`! Save yourself a lot of
    mental anguish by remembering this outright!
*   Never, ever, ever share your private key.
    [Please](http://www.google.com/search?q=site:pastebin.com+%22-----BEGIN+RSA+PRIVATE+KEY-----%22).

Generating SSH keys for keyless access
--------------------------------------

### Generating default keys

This is simple. For typical 'keyless' access, you would just hit `Enter`
when asked for a password. Not now! Choose a nice, secure password for
your default key and type it in twice.

    ssh-keygen -t rsa  
    Generating public/private rsa key pair.  
    Enter file in which to save the key (/var/user/.ssh/id_rsa):  
    Enter passphrase (empty for no passphrase):   
    Enter same passphrase again: 

You should now have two files in `.ssh` in your home folder. These are
`id_rsa`, your private key, and `id_rsa.pub`, your public key. <u>Do not
share your private key.</u>

### Copy the *Public* key to the Client

On CentOS, this is very simple:

    ssh-copy-id -i ~/.ssh/id-rsa user@client.example.com

**Note:** Don't worry; the command above doesn't copy the private key of
the identity pair. Since the command is a bash script, you can verify
this yourself.

On Mac OS X, you do it the traditional way (or you can port the
`ssh-copy-id` script over from CentOS):

    scp ~/.ssh/id_rsa.pub user@client.example.com:/var/user/.ssh/server-key.pub  
    ssh user@client.example.com  
    Password: (Enter Password)  
    cat ~/.ssh/server-key.pub >> ~/.ssh/authorized_keys  
    rm ~/.ssh/server-key.pub

### Verify Permissions

*   Make sure that `.ssh` and `authorized_keys` within it have
    permissions `700` and `600` respectively.
*   Make sure that your generated keys (public and private) have
    permissions of 600.

OpenSSH is picky like that.

### Test the Connection

Assuming you're logged in as `user` on the server:

    ssh client.example.com  
    Enter passphrase for /user/.ssh/id_rsa:

Enter the password to your SSH key, and you should be in! Yay!

Back to Square One? Not with `ssh-agent`!
-----------------------------------------

### Some background

So here's the deal: You might think this is pointless since you'll have
to enter a password anyway (for the key in this case.) However,
`ssh-agent` caches private keys (and, in our case, login information to
client machines) in memory. [From the horse's
mouth](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent):

> ssh-agent is a program to hold private keys used for public key  
> authentication (RSA, DSA).  The idea is that ssh-agent is started in the  
> beginning of an X-session or a login session, and all other windows or  
> programs are started as clients to the ssh-agent program.  Through use of  
> environment variables the agent can be located and automatically used for  
> authentication when logging in to other machines using ssh(1).

Although there exist many scripting examples which initialize and use
`ssh-agent`, we'll make that easier with [the amazing
**Keychain**](http://www.funtoo.org/Keychain). Why I like it is purely
because it maintains a single `ssh-agent` process per *system*, rather
than per *session*.

### Installation

On CentOS, just issue `yum install keychain` after installing DAG's
RPMForge repo (see Resources on this page). On OS X, check out [the
download page](http://www.funtoo.org/Keychain).

### Using Keychain

Remember that I'm trying to log in as `user` on both server and client.
I'll add this to `.bashrc` on CentOS, or `.profile` on OS X on the
client machine's user home folder:

    eval `keychain --eval --agents ssh id_rsa`

Now log out and log back into the server. You should see this:

     * keychain 2.6.10 ~ http://www.funtoo.org
     * Found existing ssh-agent: 274  
     * ssh-agent: All identities removed.  
     * Adding  1 ssh key(s): /var/user/.ssh/id_rsa  
    Enter passphrase for /var/user/.ssh/id_rsa:   
     * ssh-add: Identities added: /var/user/.ssh/id_rsa

You should have full keyless access to the client after entering the
password to the private key at this point. You may now have a drink.

### Not so fast

Remember that Keychain allows `ssh-agent` to cache private keys per
*system*. This means that the next time you log into the server,
*Keychain will not ask you for the private key's password* since it's
already cached it in memory. So if you entered the user password to the
server, here's what you get the *second* time you log in. For a
compromised server, <u>this may be as good as a password-less login</u>.

    user@server.example.com's password:   
    Last login: Sun May  2 23:53:38 2010 from 173-30-221-179.client.mchsi.com  
      
    KeyChain 2.6.8; http://www.gentoo.org/proj/en/keychain/
    Copyright 2002-2004 Gentoo Foundation; Distributed under the GPL  
      
     * Found existing ssh-agent (17986)  
     * Known ssh key: /user/.ssh/rsync-key

### Securing Keychain Further

How do you fix the situation? Simple!

1.  You assume that a user is an intruder until s/he has
    proven otherwise.
2.  If the user has not entered the correct passphrase for a private
    key, *flush the key from memory*

This works, because *keys are cleared by* `ssh-agent` *on log out, not
log in*. Here's how you'd edit `.bash_profile` or `.profile`:

    eval `keychain --clear --eval --agents ssh id_rsa`

At the next login, Keychain *will* ask you the passphrase for your
keypair. Try entering a bad one a few times and logging into the client
machine. If you've set this up right, you'll be asked for a password to
the machine!

Working with Multiple Identities
--------------------------------

In the example above, we've used the server's default SSH identity.
`id_dsa/.pub` and `id_rsa/.pub` are the keypair for this identity. It
may not be preferable to use this one single identity for multiple
clients. You're able to generate other identities using `ssh-keygen`, by
typing in the name of the keypair when it asks you for one:

    ssh-keygen -t rsa  
    Generating public/private rsa key pair.  
    Enter file in which to save the key (/var/user/.ssh/id_rsa): sample-ssh-key
    Enter passphrase (empty for no passphrase):   
    Enter same passphrase again: 

### Using the key for SSH

When you use the `id_dsa/.pub` or `id_rsa/.pub` keypairs, you don't need
to specify the `-i` (identity) switch with the SSH command since they're
default keys. This is how you'd use another identity to SSH in:

    ssh -i ~/.ssh/sample-ssh-key client2.example.com

**Note** that you used the *private* key in the example above! This
command will ask you for the password to the `sample-ssh-key` private
key, assuming you've copied over the public key to `authorized_keys` on
the client (as describe in earlier sections.)

### Using Keychain to Store Multiple Identities

Simple! For each secure identity you create, just add its appropriate
line to `.bash_profile` or `.profile` and ask Keychain to store this in
memory. For the example above:

    eval `keychain --clear --eval --agents ssh  `sample-ssh-key``

The next time you log in, you'll be asked the passphrases to *each
private key* you've defined via Keychain. For the `id_rsa` and
`sample-ssh-key` keypairs above, here's what I see:

     * keychain 2.6.10 ~ http://www.funtoo.org
     * Found existing ssh-agent: 274  
     * ssh-agent: All identities removed.  
     * Adding  1 ssh key(s): /var/user/.ssh/id_rsa  
    Enter passphrase for /var/user/.ssh/id_rsa:   
     * ssh-add: Identities added: /var/user/.ssh/id_rsa  

     * keychain 2.6.10 ~ http://www.funtoo.org
     * Found existing ssh-agent: 274  
     * ssh-agent: All identities removed.  
     * Adding  1 ssh key(s): /var/user/.ssh/sample-ssh-key  
    Enter passphrase for /var/user/.ssh/sample-ssh-key:   
     * ssh-add: Identities added: /var/user/.ssh/sample-ssh-key

All the keyless logins you've set up with multiple, passworded keys will
work now. All cron jobs will continue chugging along. Intruders won't
have direct access to clients. You'll sleep better.

Resources
---------

*   [Introduction to Keychain - Funtoo.org](http://www.funtoo.org/Keychain)
