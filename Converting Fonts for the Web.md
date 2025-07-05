## Fixing Font Families

In case they're all over the place for each style. Run with `fontforge -script fix.py`.

```python
import fontforge
import sys

fonts = [
    ("Eina02-Bold.ttf", "Bold"),
    ("Eina02-BoldItalic.ttf", "Bold Italic"),
    ("Eina02-Light.ttf", "Light"),
    ("Eina02-LightItalic.ttf", "Light Italic"),
    ("Eina02-Regular.ttf", "Regular"),
    ("Eina02-RegularItalic.ttf", "Regular Italic"),
    ("Eina02-SemiBold.ttf", "SemiBold"),
    ("Eina02-SemiboldItalic.ttf", "SemiBold Italic")
]

for filename, style in fonts:
    font = fontforge.open(filename)

    # Set family name
    font.familyname = "Eina02"
    font.fontname = filename.replace(".ttf", "")
    font.fullname = f"Eina02 {style}"

    # Set TTF names
    font.appendSFNTName("English (US)", "Family", "Eina02")
    font.appendSFNTName("English (US)", "SubFamily", style)
    font.appendSFNTName("English (US)", "Preferred Family", "Eina02")
    font.appendSFNTName("English (US)", "Preferred Styles", style)

    # Generate the corrected font
    font.generate(f"fixed_{filename}")
    font.close()
```

## Base64 Encoding

```bash
# Encode
function dataurl() {
    local MIMETYPE
    MIMETYPE=$(file -b --mime-type "$1")

    if [[ $MIMETYPE == text/* ]]; then
        MIMETYPE="${MIMETYPE};charset=utf-8";
    fi
    echo "data:${MIMETYPE};base64,$(openssl base64 -in "$1" | tr -d '\n')";
}
```

## WOFF2 &rarr; TTF

On macOS,

```bash
brew install woff2

# This will create your-font.ttf
woff2_decompress your-font.woff2
```

## The FontForge Swiss-Army Knife 🧀

Install [FontForge](https://fontforge.github.io/en-US/) (on Homebrew) and then save this as a script (`convert.pe`) and make it executable. Now you can convert whatever you'd like.

```bash
Open($1)

Generate($1:r + ".ttf")
Generate($1:r + ".otf")
Generate($1:r + ".svg")
```

Run with

```bash
fontforge -script name_of_script font.woff
```

You can also use [FontSquirrel's excellent Online Generator](https://www.fontsquirrel.com/tools/webfont-generator).


## Online Utilities

I've not had much luck with local conversions and end up using some online tool like [ConvertIO](https://convertio.co/font-converter/) or [CloudConvert](https://cloudconvert.com/).

---

## Deprecated Stuff

### EOT (Don't use this; use WOFF2)

Install `ttf2eot` via Brew and then

	ttf2eot Inconsolata.otf > Inconsolata.eot

For a bunch of files,

	for f in *.otf; do ttf2eot $f > ${f%".otf"}.eot; done

### FontConverter (Unmaintained)

Install [this via Homebrew](https://github.com/Folkloreatelier/homebrew-fonts) and convert away. It installs [this script](https://github.com/zoltan-dulac/css3FontConverter) and a bunch of dependencies.
