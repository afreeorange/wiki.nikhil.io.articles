Pre-Flight
----------

-   Speak to the sysadmin to check if the server's Apache config
    allows overrides. Basically, the `AllowOverrides` directive must be
    set to `all`.
-   Your password file can be called anything (i.e. not necessarily
    `.htpasswd`). I'm going to stick to `.htpasswd` since it's standard.

Working with Apache password files
----------------------------------

### Creating a `.htpasswd` file

Let's add Ben

` [user@example snort]# htpasswd -c .htpasswd ben`  
` New password: `  
` Re-type new password: `  
` Adding password for user ben`

### Adding more users

Vitally important to **omit the -c flag**. Not doing so will truncate
the original file!

` [user@example snort]# htpasswd .htpasswd roger`  
` New password: `  
` Re-type new password: `  
` `**`Adding`**` password for user roger`

### Removing users

Edit the `.htpasswd` file and remove the line containing the user

### Changing user passwords

Precisely the same as adding users. `htpasswd` will figure out that
you're trying to update a password:

` [user@example snort]# htpasswd .htpasswd roger`  
` New password: `  
` Re-type new password: `  
` `**`Updating`**` password for user roger`

Using `.htaccess` to tie it all together
----------------------------------------

Create a file called `.htaccess` and add the following basic options
(there are *tons* more) to use your password file:

` AuthUserFile /full/path/to/.htpasswd`  
` AuthGroupFile /dev/null`  
` AuthName "Enter your credentials to view this page"`  
` AuthType Basic`  
` `<Limit GET>  
`   require valid-user`  
` `</Limit>

Security Considerations
-----------------------

On a UNIX box, the `crypt` function is used to store passwords. I
recommend using the SHA algorithm instead:

` [user@example snort]# htpasswd -c .htpasswd ben `**`-s`**

A crucially important consideration is that *all this is done in
plaintext*. Use SSL.

Further Reading
---------------

-   [Restrict access to folders with Drupal, MySQL and
    Apache](Restrict_access_to_folders_with_Drupal,_MySQL_and_Apache "wikilink")
-   [.htaccess Authentication with
    LDAP](Subversion_Installation_and_Configuration#LDAP_integration "wikilink")



