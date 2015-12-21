`    pip install bokeh # Fail`  
`    # gevent requires libevent, although you'd never know it from the error message`  
`    brew install libevent`  
`    `  
`   # configure: error: C compiler cannot create executables ???`  
`   # Update Command Line tools for XCode`  
`   # No love`  
`   # find this: `[`http://stackoverflow.com/questions/13041525/osx-10-8-xcrun-no-such-file-or-directory`](http://stackoverflow.com/questions/13041525/osx-10-8-xcrun-no-such-file-or-directory)  
`   sudo mv  /usr/bin/xcrun /usr/bin/xcrun.bak`  
`   sudo vim /usr/bin/xcrun  # Replace contents with #!/bin/bash $@ `  
`   sudo chmod +x  /usr/bin/xcrun`  
`    `  
`   pip install gevent # NOPE`  
`   sudo ARCHFLAGS=-Wno-error=unused-command-line-argument-hard-error-in-future pip install bokeh`  
`    `  
`   # FINALLY IT WORKS`

[Source](http://www.snip2code.com/Snippet/55467/How-I-installed-Bokeh-without-conda-on-O).


[Category: OS X](Category:_OS_X "wikilink")
