Set up Dropbox to sync settings
-------------------------------

    mkdir "~/Dropbox/Settings"  
    mv ~/Library/Application\ Support/Sublime\ Text\ 2 ~/Dropbox/Settings/  
    ln -s ~/Dropbox/Settings/Sublime\ Text\ 2 ~/Library/Application\ Support/

[Can be done on Windows](http://misfoc.us/post/18018400006/syncing-sublime-text-2-settings-via-dropbox) too. Once set up, backup and symlink on other computers

    mv ~/Library/Application\ Support/Sublime\ Text\ 2 ~/Library/Application\ Support/Sublime\ Text\ 2.backup  
    ln -s ~/Dropbox/Settings/Sublime\ Text\ 2 ~/Library/Application\ Support/

Package Control
---------------

[Install this first](https://sublime.wbond.net/installation). Then hit
(or on Windows), type "Install Package", and go nuts.

Install Plugins
---------------

Some basic ones:

*   [AllAutocomplete](https://github.com/alienhard/SublimeAllAutocomplete)
*   [BracketHighlighter](https://github.com/facelessuser/BracketHighlighter)
*   [ColorHighlighter](https://github.com/Monnoroch/ColorHighlighter)
*   [SidebarEnhancements](https://github.com/titoBouzout/SideBarEnhancements)
*   [git](https://github.com/kemayo/sublime-text-2-git/wiki)
*   [GitGutter](https://github.com/jisaacks/GitGutter)
*   [SplitScreen](https://github.com/spadgos/sublime-SplitScreen)
*   [Alignment](http://wbond.net/sublime_packages/alignment)

Install Themes, Colors, and Fonts
---------------------------------

Quite a few to pick from. A few favorites:

*   (Color) [Base16](https://github.com/chriskempson/base16)
*   (Theme) [FlatLand](https://github.com/thinkpixellab/flatland)
*   (Theme) [Soda](https://github.com/buymeasoda/soda-theme/)
*   (Font) [Inconsolata](http://levien.com/type/myfonts/inconsolata.html)

Change User Preferences
-----------------------

    {
      "bold_folder_labels": true,
      "caret_style": "solid",
      "color_scheme": "Packages/Base16/base16-pop.dark.tmTheme",
      "file_exclude_patterns": [
        ".DS_Store",
        "Thumbs.db"
      ],
      "folder_exclude_patterns": [
        "bin",
        "cache",
        ".bundle",
        ".git",
        "tmp"
      ],
      "font_face": "Inconsolata",
      "font_options": [
        "subpixel_antialias"
      ],
      "font_size": 18.0,
      "highlight_line": true,
      "highlight_modified_tabs": true,
      "ignored_packages": [
        "Vintage"
      ],
      "scroll_past_end": true,
      "tab_size": 4,
      "theme": "Flatland Dark.sublime-theme",
      "translate_tabs_to_spaces": true,
      "trim_trailing_white_space_on_save": true
    }

Create a command-line shortcut (OS X)
-------------------------------------

    sudo ln -s "/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl" /usr/bin/sublime

Change the Icon
---------------

[Quite](http://flynnduism.com/alternative-icons-for-sublime-text/) [a
few](http://dribbble.com/lucifr/buckets/32936-Sublime-Text-2-Replacement-Icons)
options!

    sudo cp new_icon.icns \  
         /Applications/Sublime\ Text\ 2.app/Contents/Resources/Sublime\ Text\ 2.icns

