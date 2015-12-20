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

[vim-airline](https://github.com/bling/vim-airline)
---------------------------------------------------

To get all the nice arrows and symbols in `vim-airline`, you'll either
have to

-   [Patch your terminal
    font](https://github.com/Lokaltog/powerline-fontpatcher), or
-   [Download a pre-patched
    one](https://github.com/Lokaltog/powerline-fonts)[^1]

If you just want the symbols, [see
this](https://github.com/Lokaltog/powerline/tree/develop/font). This guy
has [some great
observations](http://www.blaenkdenum.com/posts/a-simpler-vim-statusline/)
on the amount of effort necessary to get a simple status bar. You can
always [use Sublime
Text](http://blog.andrewray.me/just-use-sublime-text/).

I stopped caring:

`   let g:airline_left_sep = ''`  
`   let g:airline_right_sep = ''`  
`   let g:airline_symbols.linenr = ''`  
`   let g:airline_symbols.branch = ''`  
`   let g:airline_symbols.paste = ''`  
`   let g:airline_symbols.whitespace = ''`

### Themes

[Here's a list](https://github.com/bling/vim-airline/wiki/Screenshots)
of included themes. Find one and set it in .vimrc with

`   let g:airline_theme = 'base16'`

### Customization

` let g:airline_section_a       (mode, paste, iminsert)`  
` let g:airline_section_b       (hunks, branch)`  
` let g:airline_section_c       (bufferline or filename)`  
` let g:airline_section_gutter  (readonly, csv)`  
` let g:airline_section_x       (tagbar, filetype, virtualenv)`  
` let g:airline_section_y       (fileencoding, fileformat)`  
` let g:airline_section_z       (percentage, line number, column number)`  
` let g:airline_section_warning (syntastic, whitespace)`

References etc
--------------

-   Look [for Unicode symbols](http://unicode-table.com/)
-   Add this to `~/.vimrc` to see the airline bar
    [https://github.com/bling/vim-airline/wiki/FAQ\#vim-airline-doesnt-appear-until-i-create-a-new-split
    1](https://github.com/bling/vim-airline/wiki/FAQ#vim-airline-doesnt-appear-until-i-create-a-new-split_1 "wikilink").

` set laststatus=2`

Footnotes
---------

<references/>
[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
[Category: Vim](Category:_Vim "wikilink")

[^1]: In iTerm2, you'll have to [change the regular and non-ASCII
    fonts](https://github.com/bling/vim-airline/issues/142) to the same
    patched one.
