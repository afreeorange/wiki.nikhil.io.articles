To illustrate, here's a directory listing:

    user@example:~/scripts/trunk/backup_scripts# ls -l  
    total 76  
    -rwxr-xr-x 1 user user   894 2011-06-27 10:00 backup.hwaddr.sh  
    -rwxr-xr-x 1 user user 12175 2011-05-02 22:54 backup.local.sh  
    -rwxr-xr-x 1 user user  6194 2011-04-12 16:53 backup.mysql.sh  
    -rwxr-xr-x 1 user user 14638 2011-04-28 11:29 backup.network.sh  
    -rwxr-xr-x 1 user user  3201 2011-05-02 09:06 backup.opendirectory.sh  
    -rwxr-xr-x 1 user user  7871 2011-04-28 11:32 backup.rotate.sh  
    -rwxr-xr-x 1 user user  3743 2011-04-12 16:53 backup.rpms.sh  
    -rwx------ 1 user user  5037 2011-06-28 15:03 backup.sh  
    -rwxr-xr-x 1 user user  4184 2011-06-27 10:01 backup.staging.sh  
    -rwxr-xr-x 1 user user  3128 2011-06-27 10:58 backup.subversion.sh

Let's say I started editing a file (e.g. `vim backup.sh`. I now hit
`Ctrl+z` to 'push' the process into the background. This is what I see:

    [1]+  Stopped                 vim backup.sh  
    user@support:~/scripts/trunk/backup_scripts#

Now I edit another file (or start another process) and hit `Ctrl+z`
again:

    [2]+  Stopped                 vim backup.subversion.sh  
    user@support:~/scripts/trunk/backup_scripts#

And so on. To list all my background jobs, I can either issue **`ps`**
or (better) **`jobs`**:

    user@support:~/scripts/trunk/backup_scripts# ps  
      PID TTY          TIME CMD  
     3561 pts/0    00:00:00 bash  
     3697 pts/0    00:00:00 vim  
     3728 pts/0    00:00:00 vim  
     3773 pts/0    00:00:00 vim  
     3790 pts/0    00:00:00 top  
     3797 pts/0    00:00:00 ps

    user@support:~/scripts/trunk/backup_scripts# jobs  
    [1]   Stopped                 vim backup.sh  
    [2]+  Stopped                 vim backup.subversion.sh  
    [3]   Stopped                 vim backup.network.sh  
    [4]-  Stopped                 top

The "+" and "-" signs indicate the first and second most recent jobs
respectively. Typing `fg` will bring the job tagged with a "+" into the
foreground. From the output above, let's say you wanted to bring `top`
into the foreground instead:

    user@support:~/scripts/trunk/backup_scripts# fg 4

To keep `top` running in the background for example ("amping off"), you
would just type `bg 4`

References
----------

-   [The Shell - The Linux Cookbook](http://www.dsl.org/cookbook/cookbook_5.html)
