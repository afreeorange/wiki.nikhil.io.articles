    # Clone the SVN repo
    git svn clone https://svn.example.com/repository
    cd trunk

    # See if all the history is fetched
    git log

    # Now add an alias for the repo
    git remote add it git@git.example.com:repository.git

    # Now (force-)push everything to the git repo
    git push it master --force
