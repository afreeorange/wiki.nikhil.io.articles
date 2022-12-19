Some assorted notes from working with Jupyter for a project.

### Config Directory

```python
import jupyter_core
jupyter_core.paths.jupyter_config_dir()
```

That gave me `/Users/nikhil/.jupyter` on my M1 Mac (i.e., `$HOME/.jupyter`)

### Custom CSS (and JS!)

You would add your custom CSS and JS to these files:

```
$HOME/.jupyter/custom/custom.css
$HOME/.jupyter/custom/custom.js
```

Note that this is for the web interface that you get from running `jupyter-lab`. [The VSCode plugin](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter#review-details) (which is far superior to the web interface) _will not_ respect these settings!

