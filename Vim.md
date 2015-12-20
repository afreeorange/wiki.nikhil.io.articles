Tricks
------

-   [Reload `.vimrc`](http://superuser.com/a/132030) without restarting
    vim:

` :so ~/.vimrc`  
` :so $MYVIMRC`  
` :so %`

:   The last command is for when you're editing it.

Package management
------------------

[Pathogen](https://github.com/tpope/vim-pathogen) seem slike a

Color Schemes
-------------

### Installation

As an example, I'll install the [Base16
Eighties](http://chriskempson.github.io/base16/#eighties) theme.

    mkdir ~/.vim/colors
    curl https://raw.githubusercontent.com/chriskempson/base16-vim/master/colors/base16-eighties.vim > ~/.vim/colors/base16-eighties.vim

Fire up vim, then `:colorscheme base16-eighties` (tab-completion works!)
and voila. Add your favorite to `~/.vimrc`

### Finding Themes

-   [Vivify](http://bytefluent.com/vivify/) is a nice GUI.
    [Villustrator](http://www.villustrator.com/) is another.
-   Base16: [themes](https://github.com/chriskempson/base16-vim) and
    [preview](http://chriskempson.github.io/base16). You can also [make
    your own](https://github.com/chriskempson/base16-builder).

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Vim](Category:_Vim "wikilink")
