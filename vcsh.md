Using [vcsh](https://github.com/RichiH/vcsh) for when I set up a new computer

```bash
vcsh init dotfiles
vcsh dotfiles remote add origin git@github.com:afreeorange/dotfiles.git
rm ~/.bash_profile
rm ~/.editorconfig
rm ~/.gitconfig
rm ~/.gitignore
rm ~/.gitignore
rm ~/.sublime_packages
rm ~/.sublimerc
rm ~/.vimrc
vcsh dotfiles pull origin master
```
