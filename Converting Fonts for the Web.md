I started with an OpenType file.

### TTF and SVG

Install [FontForge](https://fontforge.github.io/en-US/) (on Homebrew) and then save this as a script

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

### EOT

Install `ttf2eot` via Brew and then

	ttf2eot Inconsolata.ttf > Inconsolata.eot

