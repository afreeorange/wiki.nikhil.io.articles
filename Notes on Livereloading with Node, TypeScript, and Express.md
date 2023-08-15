Quick notes I'll revise later.

### Required Packages

- `livereload`
- `tsx` or `nodemon` + `ts-node`

### TSX

```bash
# Add this snippet to the rendered template.
<script src="//localhost:35729/livereload.js?snipver=1"></script>

# Start this process
npx livereload src/ -e 'tsx,ts' -w 1000

# And then this
tsx watch .
```

This will be fine. Not using a 1s delay will make things look 'stuck'.

### Nodemon + `ts-node`

Add this to your Express App

```javascript
import livereload from "livereload";
import connectLiveReload from "connect-livereload";

if (process.env.NODE_ENV === "development") {
  const liveReloadServer = livereload.createServer();

  liveReloadServer.watch(path.join(__dirname, "src"));

  liveReloadServer.server.once("connection", () => {
    setTimeout(() => {
      liveReloadServer.refresh("/");
    }, 10);
  });

  app.use(connectLiveReload());
}
```

```bash
NODE_ENV=development npx nodemon --exec ts-node src/index.tsx
```

This will work as well. Slower with the defaults.
