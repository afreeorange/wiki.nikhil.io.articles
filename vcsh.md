    vcsh init dotfiles
    vcsh dotfiles remote add origin https://github.com/afreeorange/dotfiles.git
    rm ~/.bash_profile
    rm ~/.editorconfig
    rm ~/.gitconfig
    rm ~/.gitignore
    rm ~/.gitignore
    rm ~/.sublime_packages
    rm ~/.sublimerc
    rm ~/.vimrc
    vcsh dotfiles branch -u origin/master master
    vcsh dotfiles pull origin master