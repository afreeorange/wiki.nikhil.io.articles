```python
import tornado.httpserver
import tornado.ioloop
import tornado.concurrent
from tornado.options import (
    define,
    options,
    parse_command_line
)
import tornado.web


class SyncHandler(tornado.web.RequestHandler):
    """Simple request handler that blocks the IOLoop
    """
    def get(self, seconds):
        blocking_task(int(seconds))
        self.write("Blocked {}".format(seconds))

class AsyncHandler(tornado.web.RequestHandler):
    """Use Tornado's coroutine generators to handle an incoming request.
       A slow, blocking function is submitted to an execution pool and
       control is released back to the Tornado server's IOLoop to
       service other incoming requests.
    """
    # Create a thread pool
    import concurrent
    executor = concurrent.futures.ThreadPoolExecutor(10)

    @tornado.concurrent.run_on_executor
    def blocking_task(self, n=10):
        from time import sleep
        print("Sleeping {}".format(n))
        sleep(n)
        return n

    @tornado.gen.coroutine
    def get(self, seconds):
        yield self.blocking_task(int(seconds))
        self.write("Blocked {}".format(seconds))

class IndexHandler(tornado.web.RequestHandler):
    """Handle the index page
    """
    def get(self):
        self.write("I'm a Tornado server")

define("port", default=8000, help="Port to run Tornado instance on")
define("reload", default=False, help="Reload server on change")
define("debug", default=False, help="Show debug logs")

if __name__ == '__main__':
    parse_command_line()

    tornado.httpserver.HTTPServer(
        tornado.web.Application(
                    handlers=[
                        (r"/", IndexHandler),
                        (r"/sync\/?(\d+)", SyncHandler),
                        (r"/async\/?(\d+)", AsyncHandler)
                    ],
                    autoreload=options.reload,
                    debug=options.debug,
                )
        ).listen(options.port)

    tornado.ioloop.IOLoop.instance().start()
```

---

```python
import tornado.httpserver
import tornado.ioloop
from tornado.options import (
    define,
    options,
    parse_command_line
)
import tornado.web


# Create a thread pool
import concurrent
executor = concurrent.futures.ThreadPoolExecutor(10)

def blocking_task(n=10):
    from time import sleep
    print("Sleeping {}".format(n))
    sleep(n)

class SyncHandler(tornado.web.RequestHandler):
    """Simple request handler that blocks the IOLoop
    """
    def get(self, seconds):
        blocking_task(int(seconds))
        self.write("Blocked {}".format(seconds))

class AsyncHandler(tornado.web.RequestHandler):
    """Use Tornado's coroutine generators to handle an incoming request.
       A slow, blocking function is submitted to an execution pool and
       control is released back to the Tornado server's IOLoop to
       service other incoming requests.

       There's also a @run_on_executor decorator that can be applied
       to the blocking function to prettify things.
       E.g. https://gist.github.com/methane/2185380#comment-1301483

       References
       ----------
       * http://tornado.readthedocs.org/en/latest/guide/coroutines.html
       * http://blog.trukhanov.net/Running-synchronous-code-on-tornado-asynchronously/
       * http://lbolla.info/blog/2013/01/22/blocking-tornado
    """
    @tornado.gen.coroutine
    def get(self, seconds):
        yield executor.submit(blocking_task, int(seconds))
        self.write("Blocked {}".format(seconds))

class IndexHandler(tornado.web.RequestHandler):
    """Handle the index page
    """
    def get(self):
        self.write("I'm a Tornado server")

define("port", default=8000, help="Port to run Tornado instance on")
define("reload", default=False, help="Reload server on change")
define("debug", default=False, help="Show debug logs")

if __name__ == '__main__':
    parse_command_line()

    tornado.httpserver.HTTPServer(
        tornado.web.Application(
                    handlers=[
                        (r"/", IndexHandler),
                        (r"/sync\/?(\d+)", SyncHandler),
                        (r"/async\/?(\d+)", AsyncHandler)
                    ],
                    autoreload=options.reload,
                    debug=options.debug,
                )
        ).listen(options.port)

    tornado.ioloop.IOLoop.instance().start()
```
