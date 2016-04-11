I started with an OpenType file.

### TTF and SVG

Install [FontForge](https://fontforge.github.io/en-US/) (on Homebrew) and then save this as a script (`convert.pe`) and make it executable.

```bash
Open($1)
Generate($1:r + ".ttf")
Generate($1:r + ".svg")
```

Run with

    fontforge -script name_of_script font.otf

### WOFF

Use [this tool](http://people.mozilla.org/~jkew/woff/) (not on Homebrew) called `sfnt2woff`

    sfnt2woff font.otf

For a bunch of 'em,

    for f in *.otf; do ~/Downloads/sfnt2woff $f; done

### EOT

Install `ttf2eot` via Brew and then

	ttf2eot Inconsolata.otf > Inconsolata.eot

For a bunch of files,

	for f in *.otf; do ttf2eot $f > ${f%".otf"}.eot; done

