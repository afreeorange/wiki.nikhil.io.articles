Tricks
------

*   [Reload `.vimrc`](http://superuser.com/a/132030) without restarting vim:

        :so ~/.vimrc  
        :so $MYVIMRC  
        :so %

:   The last command is for when you're editing it.

Package management
------------------

[Vundle](https://github.com/gmarik/Vundle.vim) FTW.

    " Required Vundle stuff  
    set rtp+=~/.vim/bundle/Vundle.vim  
    call vundle#begin()  
    Plugin 'gmarik/Vundle.vim'  
      
    " Plugins  
    Plugin 'airblade/vim-gitgutter'  
    Plugin 'scrooloose/nerdtree'  
    Plugin 'chriskempson/base16-vim'  
    Plugin 'bling/vim-airline'  
    Plugin 'tpope/vim-fugitive'  
    Plugin 'jmcantrell/vim-virtualenv'  
    Plugin 'bling/vim-bufferline'  
      
    " End Vundle stuff  
    call vundle#end()  
    filetype plugin indent on

Color Schemes
-------------

iTerm2, Base16 theme on OS X. Need do three things:

*   Install [relevant iTerm2
    theme](https://github.com/chriskempson/base16-iterm2)
*   Set up the [Base16
    shell](https://github.com/chriskempson/base16-shell) in \~/.profile
*   Install [Base16 vim
    themes](https://github.com/chriskempson/base16-vim) (done
    via Vundle)

Then,

    let base16colorspace=256  
    let g:airline_theme='base16'  
    colorscheme base16-railscasts

[vim-airline](https://github.com/bling/vim-airline)
---------------------------------------------------

To get all the nice arrows and symbols in `vim-airline`, you'll either
have to

*   [Patch your terminal
    font](https://github.com/Lokaltog/powerline-fontpatcher), or
*   [Download a pre-patched
    one](https://github.com/Lokaltog/powerline-fonts)[^1]

If you just want the symbols, [see this](https://github.com/Lokaltog/powerline/tree/develop/font). This guy has [some great observations](http://www.blaenkdenum.com/posts/a-simpler-vim-statusline/)
on the amount of effort necessary to get a simple status bar. You can always 
[use Sublime Text](http://blog.andrewray.me/just-use-sublime-text/).

I stopped caring (see source)

    let g:airline_left_sep = ''  
    let g:airline_right_sep = ''  
    let g:airline_symbols.linenr = ''  
    let g:airline_symbols.branch = ''  
    let g:airline_symbols.paste = ''  
    let g:airline_symbols.whitespace = ''

### Themes

[Here's a list](https://github.com/bling/vim-airline/wiki/Screenshots)
of included themes. Find one and set it in .vimrc with

    let g:airline_theme = 'base16'

### Customization

    let g:airline_section_a       (mode, paste, iminsert)  
    let g:airline_section_b       (hunks, branch)  
    let g:airline_section_c       (bufferline or filename)  
    let g:airline_section_gutter  (readonly, csv)  
    let g:airline_section_x       (tagbar, filetype, virtualenv)  
    let g:airline_section_y       (fileencoding, fileformat)  
    let g:airline_section_z       (percentage, line number, column number)  
    let g:airline_section_warning (syntastic, whitespace)

References etc
--------------

*   Look [for Unicode symbols](http://unicode-table.com/)
*   Add this to `~/.vimrc` to see the airline bar ([source](https://github.com/bling/vim-airline/wiki/FAQ\#vim-airline-doesnt-appear-until-i-create-a-new-split))

        set laststatus=2

*   [Vivify](http://bytefluent.com/vivify/) is a nice theme-making GUI.
    [Villustrator](http://www.villustrator.com/) is another.
*   You can also [make your own](https://github.com/chriskempson/base16-builder) 
    Base16 theme.
*   Some [good notes](http://blog.tjll.net/yet-another-vim-setup/) on a
    basic Vim setup.

Footnotes
---------

[^1]: In iTerm2, you'll have to [change the regular and non-ASCII
    fonts](https://github.com/bling/vim-airline/issues/142) to the same
    patched one.
