### Encode and Decode

#### Encode

```bash
function dataurl() {
    local MIMETYPE
    MIMETYPE=$(file -b --mime-type "$1")

    if [[ $MIMETYPE == text/* ]]; then
        MIMETYPE="${MIMETYPE};charset=utf-8";
    fi
    echo "data:${MIMETYPE};base64,$(openssl base64 -in "$1" | tr -d '\n')";
}
```

#### Decode

```bash
#!/bin/bash

ENCODED_FILE=$1
sed 's/data:application\/font-woff;base64,//g' "$ENCODED_FILE" | base64 -D > "${ENCODED_FILE%%.*}.woff"
```

### WOFF, WOFF2, TTF, OTF, and SVG

Install [FontForge](https://fontforge.github.io/en-US/) (on Homebrew) and then save this as a script (`convert.pe`) and make it executable. Now you can convert whatever you'd like.

```bash
Open($1)

Generate($1:r + ".ttf")
Generate($1:r + ".otf")
Generate($1:r + ".svg")
```

Run with

    fontforge -script name_of_script font.woff

### Deprecated Stuff

#### EOT (Deprecated Format)

Install `ttf2eot` via Brew and then

	ttf2eot Inconsolata.otf > Inconsolata.eot

For a bunch of files,

	for f in *.otf; do ttf2eot $f > ${f%".otf"}.eot; done

#### Utilities

Install [this via Homebrew](https://github.com/Folkloreatelier/homebrew-fonts) and convert away. It installs [this script](https://github.com/zoltan-dulac/css3FontConverter) and a bunch of dependencies.
