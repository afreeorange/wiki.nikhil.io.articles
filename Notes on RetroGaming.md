Got myself [a tiny 8BitDo controller](https://a.co/d/2eNjym2) for use with the excellent [OpenEmu for macOS](https://openemu.org/) (can't believe it's free).

Many ROM sites appear to be broken but [Vimm's Lair](https://vimm.net/) appears to be active and offer quite a few. There's also the Reddit [/r/ROMS community](https://old.reddit.com/r/Roms/).

### Karabiner Remappings

Strictly optional.

The controller registers itself as a keyboard. I use the excellent Karabiner to remap the buttons so I don't have to remember which letter, for example, the "Up Arrow" on the controller is (it's `c`). Here's my config. It goes into a list under `profiles` -> `devices`.

I also used the `-` and `+` buttons at the top for `Select` and `Start` respectively.

```json
{
"identifiers": {
    "is_keyboard": true,
    "product_id": 36897,
    "vendor_id": 11720
},
"simple_modifications": [
    {
        "from": { "key_code": "e" },
        "to": [{ "key_code": "left_arrow" }]
    },
    {
        "from": { "key_code": "f" },
        "to": [{ "key_code": "right_arrow" }]
    },
    {
        "from": { "key_code": "c" },
        "to": [{ "key_code": "up_arrow" }]
    },
    {
        "from": { "key_code": "d" },
        "to": [{ "key_code": "down_arrow" }]
    },
    {
        "from": { "key_code": "g" },
        "to": [{ "key_code": "a" }]
    },
    {
        "from": { "key_code": "j" },
        "to": [{ "key_code": "b" }]
    },
    {
        "from": { "key_code": "h" },
        "to": [{ "key_code": "x" }]
    },
    {
        "from": { "key_code": "i" },
        "to": [{ "key_code": "y" }]
    }
]
}
```
