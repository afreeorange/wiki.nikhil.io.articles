Assumes that `vcsh` is set up with my [dotfiles](https://github.com/afreeorange/dotfiles).

```bash
[[ "$(uname)" -ne "Darwin" ]] && echo "This is not a Mac" && exit 1

sudo ln -s  "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/sublime

wget -O ~/Library/Application\ Support/Sublime\ Text\ 3/Installed\ Packages/Package\ Control.sublime-packagea https://packagecontrol.io/Package%20Control.sublime-package

rm ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User/Package\ Control.sublime-settings
ln -s ~/.sublime_packages ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User/Package\ Control.sublime-settings

rm ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User/Preferences.sublime-settings
ln -s ~/.sublimerc ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User/Preferences.sublime-settings
```
