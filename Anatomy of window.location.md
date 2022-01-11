If this is the URI:

```
http://localhost:4200/landing?query=1#2
```

These are its constituents in the `window.location` object:

```javascript
window.location.hash: "#2"
window.location.host: "localhost:4200"
window.location.hostname: "localhost"
window.location.href: "http://localhost:4200/landing?query=1#2"
window.location.origin: "http://localhost:4200"
window.location.pathname: "/landing"
window.location.port: "4200"
window.location.protocol: "http:"
window.location.search: "?query=1"
```

[Via](https://stackoverflow.com/a/54691801)

