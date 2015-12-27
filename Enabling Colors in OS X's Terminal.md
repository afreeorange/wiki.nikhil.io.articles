Add these to `.bash_profile`

    export CLICOLOR=1  
    export LSCOLORS=ExFxCxDxBxegedabagacad

Remove any mention of `xterm`

Other info
----------

The `LSCOLORS` variable needs 11 letters for colors.

### Colors

    a  black  
    b  red  
    c  green  
    d  brown  
    e  blue  
    f  magenta  
    c  cyan  
    h  light grey  
    A  block black, usually shows up as dark grey  
    B  bold red  
    C  bold green  
    D  bold brown, usually shows up as yellow  
    E  bold blue  
    F  bold magenta  
    G  bold cyan  
    H  bold light grey; looks like bright white  
    x  default foreground or background

### Order

1.  directory
2.  symbolic link
3.  socket
4.  pipe
5.  executable
6.  block special
7.  character special
8.  executable with setuid bit set
9.  executable with setgid bit set
10. directory writable to others, with sticky bit
11. directory writable to others, without sticky bit
