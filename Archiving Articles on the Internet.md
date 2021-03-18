Looked at three options

* [SingleFile](https://github.com/gildas-lormeau/SingleFile) (there's [a CLI](https://github.com/gildas-lormeau/SingleFile/blob/master/cli/README.MD#manual-installation))
* [ArchiveBox](https://archivebox.io/)
* [`readability-cli`](https://gitlab.com/gardenappl/readability-cli) which wraps [Readability](https://github.com/mozilla/readability)

Requirement: I don't need any JavaScript in my archive. Don't really care about the images either. Just the text.

### SingleFile

Not bad at all. Everything but the JavaScript, all scrunched into a... single file.

```bash
# Use JsDOM instead of Chrome/Puppeteer to avoid JavaScript
npm i -g jsdom
npm i -g "gildas-lormeau/SingleFile#master"
```

### ArchiveBox

Saves _everything_ kinda like archive.is. Images, CSS, JS, fonts, everything.

```bash
# On macOS
brew install archivebox/archivebox/archivebox

# Get the Readability driver
npm install --prefix . "git+https://github.com/ArchiveBox/ArchiveBox.git"

archivebox init
archivebox add https://www.washingtonpost.com/politics/2021/01/15/pillow-salesman-apparently-has-some-ideas-about-declaring-martial-law/?utm_source=reddit.com
archivebox server
```

### `readability-cli`

Saves _just_ the DOM. No styling. [Example](https://static-log.nikhil.io/c/collatz-in-ts.html).

### Result

ArchiveBox is awesome but I ended up using SingleFile for a good balance. Plus, 
