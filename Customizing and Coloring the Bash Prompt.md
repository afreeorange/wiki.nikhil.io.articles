The Prompt Variable
-------------------

```bash
user@ubuntu:~# echo $PS1 
${debian_chroot:+($debian_chroot)}\\[\033[01;32m\\]\u@\h\\[\033[00m\\]:\\[\033[01;34m\\]\w\\[\03300m\\]\$
```

Making Sense of the Above
-------------------------

### Flags

|  Flag  |                                                  What it shows                                                   |
|--------|------------------------------------------------------------------------------------------------------------------|
| `\a`   | ASCII bell character (07)                                                                                        |
| `\d`   | date in "Weekday Month Date" format (e.g., "Tue May 26")                                                         |
| `\e`   | ASCII escape character (033)                                                                                     |
| `\h`   | hostname up to the first `.’                                                                                     |
| `\H`   | hostname                                                                                                         |
| `\n`   | newline                                                                                                          |
| `\r`   | return                                                                                                           |
| `\s`   | name of the shell, the basename of $0 (the portion following the final slash)                                    |
| `\t`   | current time in 24-hour HH:MM:SS format                                                                          |
| `\T`   | current time in 12-hour HH:MM:SS format                                                                          |
| `\@`   | current time in 12-hour am/pm format                                                                             |
| `\u`   | username of the current user                                                                                     |
| `\v`   | version of bash (e.g., 2.00)                                                                                     |
| `\V`   | release of bash, version + patchlevel (e.g., 2.00.0)                                                             |
| `\w`   | current working directory                                                                                        |
| `\W`   | basename of the current working direc-tory                                                                       |
| `\!`   | history number of this command                                                                                   |
| `\#`   | command number of this command                                                                                   |
| `\$`   | the effective UID is 0, a #, otherwise a $                                                                       |
| `\nnn` | character corresponding to the octal number nnn                                                                  |
| `\\`   | backslash                                                                                                        |
| `\[`   | a sequence of non-printing characters, which could be used to embed a terminal con-trol sequence into the prompt |
| `\]`   | a sequence of non-printing characters                                                                            |

### Colors

Here's a color table.

|    Color     |  Code  |
|--------------|--------|
| Black        | `0;30` |
| Dark Gray    | `1;30` |
| Blue         | `0;34` |
| Light Blue   | `1;34` |
| Green        | `0;32` |
| Light Green  | `1;32` |
| Cyan         | `0;36` |
| Light Cyan   | `1;36` |
| Red          | `0;31` |
| Light Red    | `1;31` |
| Purple       | `0;35` |
| Light Purple | `1;35` |
| Brown        | `0;33` |
| Yellow       | `1;33` |
| Light Gray   | `0;37` |
| White        | `1;37` |

To color anything,

* Prefix with `[33[<Color>m\]`
* Suffix with `\[33[00m\]`

For example, to color the directory's basename (`\W`) yellow (`1;33`),
you'd have this:

```bash
# Exploded to illustrate  
[33[01;33m\] \W \\[33[00m\\]  
  
# Together  
[33[01;33m\]\W\\[33[00m\\]  
```
