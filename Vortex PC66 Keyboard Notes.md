- [TOC](#toc)
  - [Bluetooth](#bluetooth)
  - [Shortcuts](#shortcuts)
    - [Navigation and System Controls](#navigation-and-system-controls)
    - [Arrow Keys](#arrow-keys)
    - [Special Functions](#special-functions)
    - [Layout and OS Switching](#layout-and-os-switching)
    - [Factory Restore](#factory-restore)
  - [Problems](#problems)
    - [Missing Tilde and Backtick :/](#missing-tilde-and-backtick-)
    - [Connecting Wirelessly](#connecting-wirelessly)

---

[Here's the manual](/assets/PC66_Manual.pdf).

## Bluetooth

1. <kbd>Fn</kbd> + <kbd>Home</kbd> until LED is solid blue.
2. Now press any one of these to enter Pairing Mode.
   - <kbd>PgUp</kbd> for Device 1
   - <kbd>PgDn</kbd> for Device 2
   - <kbd>End</kbd> for Device 3

Youc an now use <kbd>Fn</kbd> + <kbd>PgUp</kbd>, <kbd>PgDn</kbd>, or <kbd>End</kbd> to switch between devices

## Shortcuts

- <kbd>Fn</kbd> + <kbd>Esc</kbd> = <kbd>`</kbd>
- <kbd>Fn</kbd> + <kbd>Shift</kbd> + <kbd>Esc</kbd> = <kbd>~</kbd>
- <kbd>Fn</kbd> + <kbd>1</kbd> = <kbd>F1</kbd>
- <kbd>Fn</kbd> + <kbd>2</kbd> = <kbd>F2</kbd>
- <kbd>Fn</kbd> + <kbd>3</kbd> = <kbd>F3</kbd>
- <kbd>Fn</kbd> + <kbd>4</kbd> = <kbd>F4</kbd>
- <kbd>Fn</kbd> + <kbd>5</kbd> = <kbd>F5</kbd>
- <kbd>Fn</kbd> + <kbd>6</kbd> = <kbd>F6</kbd>
- <kbd>Fn</kbd> + <kbd>7</kbd> = <kbd>F7</kbd>
- <kbd>Fn</kbd> + <kbd>8</kbd> = <kbd>F8</kbd>
- <kbd>Fn</kbd> + <kbd>9</kbd> = <kbd>F9</kbd>
- <kbd>Fn</kbd> + <kbd>0</kbd> = <kbd>F10</kbd>
- <kbd>Fn</kbd> + <kbd>-</kbd> = <kbd>F11</kbd>
- <kbd>Fn</kbd> + <kbd>=</kbd> = <kbd>F12</kbd>

### Navigation and System Controls

- <kbd>Fn</kbd> + <kbd>Backspace</kbd> = <kbd>Delete</kbd>
- <kbd>Fn</kbd> + <kbd>;</kbd> = <kbd>Insert</kbd>
- <kbd>Fn</kbd> + <kbd>P</kbd> = <kbd>Print</kbd>
- <kbd>Fn</kbd> + <kbd>[</kbd> = <kbd>Scroll</kbd>
- <kbd>Fn</kbd> + <kbd>]</kbd> = <kbd>Pause</kbd>
- <kbd>Fn</kbd> + <kbd>A</kbd> = <kbd>Previous</kbd>
- <kbd>Fn</kbd> + <kbd>S</kbd> = <kbd>Play/Pause</kbd>
- <kbd>Fn</kbd> + <kbd>D</kbd> = <kbd>Next</kbd>
- <kbd>Fn</kbd> + <kbd>F</kbd> = <kbd>Mute</kbd>
- <kbd>Fn</kbd> + <kbd>G</kbd> = <kbd>Volume -</kbd>
- <kbd>Fn</kbd> + <kbd>H</kbd> = <kbd>Volume +</kbd>

### Arrow Keys

- <kbd>Fn</kbd> + <kbd>I</kbd> = <kbd>Up</kbd>
- <kbd>Fn</kbd> + <kbd>J</kbd> = <kbd>Left</kbd>
- <kbd>Fn</kbd> + <kbd>K</kbd> = <kbd>Down</kbd>
- <kbd>Fn</kbd> + <kbd>L</kbd> = <kbd>Right</kbd>

### Special Functions

- <kbd>Fn</kbd> + <kbd>?</kbd> = <kbd>66key:Menu</kbd> / <kbd>68key:Windows</kbd>
- <kbd>Fn</kbd> + <kbd>></kbd> = <kbd>Windows_R</kbd>
- <kbd>Fn</kbd> + <kbd>Z</kbd> = <kbd>66key:Windows</kbd> / <kbd>68key:Menu</kbd>
- <kbd>Fn</kbd> + <kbd>Win</kbd> = <kbd>68key:Winlock</kbd>
- <kbd>Fn</kbd> + <kbd>Spacebar</kbd> = <kbd>Capslock</kbd>

### Layout and OS Switching

(LED will flash a white color)

- <kbd>Fn</kbd> + any <kbd>Alt</kbd> + <kbd>PgUp</kbd> (press 2s) = <kbd>Colemak</kbd>
- <kbd>Fn</kbd> + any <kbd>Alt</kbd> + <kbd>PgDn</kbd> (press 2s) = <kbd>Dvorak</kbd>
- <kbd>Fn</kbd> + any <kbd>Alt</kbd> + <kbd>End</kbd> (press 2s) = <kbd>Qwerty</kbd>
- <kbd>Fn</kbd> + any <kbd>Alt</kbd> + <kbd>HOME</kbd> (press 2s) = <kbd>Switch OS</kbd>

### Factory Restore

<kbd>Fn</kbd> + <kbd>Alt_L</kbd> + <kbd>Alt_R</kbd> (press for 3s)

## Problems

### Missing Tilde and Backtick :/

The I use the tilde <kbd>~</kbd> and backtick <kbd>`</kbd> keys a lot. They're _physically_ missing on this keyboard and it's a PITA. On macOS, this means that I cannot use <kbd>Command</kbd>+<kbd>`</kbd> to "move focus to the next window" (i.e. cycling between instances of the same app).

The solution was the lovely Karabiner 🥰 Since I use <kbd>Esc</kbd> so little, I mapped it to tilde/backtick. Then mapped <kbd>Ctrl</kbd>+<kbd>Esc</kbd> to the _real_ <kbd>Esc</kbd>. Pure Value™. Here's the config:

```json
{
  "profiles": [
    {
      "name": "Default profile",
      "selected": true,
      "virtual_hid_keyboard": { "keyboard_type_v2": "ansi" },
      "complex_modifications": {
        "rules": [
          {
            "description": "Map Esc to ~ and `",
            "manipulators": [
              {
                "type": "basic",
                "from": {
                  "key_code": "escape"
                },
                "to": [
                  {
                    "key_code": "grave_accent_and_tilde"
                  }
                ]
              }
            ]
          },
          {
            "description": "Map Ctrl+Esc to Esc",
            "manipulators": [
              {
                "type": "basic",
                "from": {
                  "key_code": "escape",
                  "modifiers": {
                    "mandatory": ["control"]
                  }
                },
                "to": [
                  {
                    "key_code": "escape"
                  }
                ]
              }
            ]
          },
          {
            "description": "Remap (Command + Esc) to (Command + Backtick) so you can 'Move focus to next window'",
            "manipulators": [
              {
                "type": "basic",
                "from": {
                  "key_code": "escape",
                  "modifiers": {
                    "mandatory": ["command"]
                  }
                },
                "to": [
                  {
                    "key_code": "grave_accent_and_tilde",
                    "modifiers": ["command"]
                  }
                ]
              }
            ]
          }
        ]
      }
    }
  ]
}
```

### Connecting Wirelessly

When _not_ connected via a cable, macOS will think this is an Apple-made Keyboard. [This has apparently been a problem since macOS Ventura](https://discussions.apple.com/thread/254485490?sortBy=rank). The Karabiner config above addresses this issue as well 🌸
