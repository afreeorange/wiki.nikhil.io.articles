A stupid-simple, barebones setup using:

* `poetry` for virtual environments
* `black` for formatting and `isort` for import sorting
* `flake8` for linting
* `pytest` for testing
* `mypy` for type-checking
* (Optional but recommended) `pylance` for IntelliSense™
* (Optional but recommended) `rewrap` extension for docstrings and comments

**Written for macOS** since that's what _most_ of our devs use. Paths and shortcuts will vary for Windows and Linux.

Using this guide will help VSCode warn you of problematic code via red squiggles. You can then get an explanation via the "Problems" panel (<kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>m</kbd>.)

### In a Terminal

```bash
# Install poetry globally in your Python (which you should
# manage with pyenv)
pip install poetry

# Create a new project
poetry new my_project

# Install these as development dependencies. Which they are.
poetry add black flake8 isort pytest --dev
```

### In VSCode

#### Tell VSCode where all the virtual environments are

On a Mac, type <kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>p</kbd> to pull up the [Command Palette](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette). Search for `python.venvPath` and set it to

    ~/Library/Caches/pypoetry/virtualenvs

#### Set the Python Interpreter to point at the Virtual Environment

Type "Python Interpreter" in the Command Palette, hit enter, and pick something that looks like `my_project-33ebFI0f-py3.9`. You can see _exactly_ which one by typing this in the terminal:

```bash
poetry env info -p
```

For the next steps, and on a Mac, type <kbd>Cmd</kbd>+<kbd>,</kbd> to open up your Editor Settings.

#### Use `black`

Search for `python.formatting.provider` and choose black. 

You might like to format automagically on save. To do this, search for `editor.formatOnSave`.

#### Use `isort`

Nothing to do! <kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>p</kbd> and type "sort imports".

#### Use `flake8`

Search for `python.linting.flake8Enabled` and enable Flake8.

#### Use `pytest`

Search for `python.testing.pytestEnabled` and enable PyTest. You'll see a new "Beaker" icon in the Activity sidebar. Click it to see all the tests. VSCode has pretty powerful testing setup and discovery options [you might be interested in](https://code.visualstudio.com/docs/python/testing).

#### Use `mypy`

Search for `python.linting.mypyEnabled` and enable MyPy.

---

The setup above should give you a nice Borg-like experience with Python development but you can make things even better using proprietary closed-source software!

#### PyLance

This is a language server for Python and is the intelligence in IntelliSense.

To install, just get it from the VSCode Marketplace. It won't work out of the box until you add this to `.vscode/settings.json`[1] in your project root.

```
{
    "python.analysis.useImportHeuristic": true
}
```

Now enjoy the awesome benefits of a language server. E.g. If you're on macOS and a symbol doesn't resolve, <kbd>Cmd</kbd>+<kbd>.</kbd> is your friend.

**Note**: That setting above is in beta so you may have to type "Restart: Python Language Server" once in a while in the Command Palette if you get yellow squigglies that don't go away EVEN THOUGH _YOU_ KNOW _YOU_ RIGHT, BOO 😤

#### ReWrap

[Install this extension](https://marketplace.visualstudio.com/items?itemName=stkb.rewrap). Then make sure you have `79` in the array of rulers in `editor.rulers` in your VSCode Settings. Now (on a Mac) press <kbd>Option</kbd>+<kbd>q</kbd> to rewrap (ha) comments to fit into 79 columns.

#### And Finally!

Exclude some unnecessary things in your sidebar. Pull up your VSCode settings (<kbd>Command</kbd>+<kbd>,</kbd> on macOS) and add these patterns

```
**/__pycache__
**/.pytest_cache
```

---

[1]: DO NOT CHECK THAT FILE AND FOLDER INTO SOURCE CONTROL.
