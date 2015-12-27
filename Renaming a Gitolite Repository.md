*   Rename the repo on the server

    ```bash
    ssh gitserver  
      
    # This is usually /home/git/repositories  
    cd $REPO_BASE  
      
    mv old_repo new_repo
    ```

*   Rename the repo in `gitolite.conf` and push to server.

Fin.
