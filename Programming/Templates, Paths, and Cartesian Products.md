Was working on a CLI application at work that required a lot of 'deep' interpolation. Was pretty satisfied with this result.

```javascript
// Use this...
const variables = {
  environments: ["production", "development", "testing"],
  regions: ["us-east-1", "sa-east-1"],
  stages: ["alpha", "beta", "rc", "live"],
  appName: "foo",
};

// ... to interpolate this...
const templateName = `config.{{stages}}.LOLZILLA.{{appName}}__{{environments}}--{{regions}}.json`;

// ... to yield this:
const expectedOutput = [
  "config.alpha.LOLZILLA.foo__production--us-east-1.json",
  "config.alpha.LOLZILLA.foo__development--us-east-1.json",
  "config.alpha.LOLZILLA.foo__testing--us-east-1.json",
  "config.alpha.LOLZILLA.foo__production--sa-east-1.json",
  "config.alpha.LOLZILLA.foo__development--sa-east-1.json",
  "config.alpha.LOLZILLA.foo__testing--sa-east-1.json",
  "config.beta.LOLZILLA.foo__production--us-east-1.json",
  "config.beta.LOLZILLA.foo__development--us-east-1.json",
  "config.beta.LOLZILLA.foo__testing--us-east-1.json",
  "config.beta.LOLZILLA.foo__production--sa-east-1.json",
  "config.beta.LOLZILLA.foo__development--sa-east-1.json",
  "config.beta.LOLZILLA.foo__testing--sa-east-1.json",
  "config.rc.LOLZILLA.foo__production--us-east-1.json",
  "config.rc.LOLZILLA.foo__development--us-east-1.json",
  "config.rc.LOLZILLA.foo__testing--us-east-1.json",
  "config.rc.LOLZILLA.foo__production--sa-east-1.json",
  "config.rc.LOLZILLA.foo__development--sa-east-1.json",
  "config.rc.LOLZILLA.foo__testing--sa-east-1.json",
  "config.live.LOLZILLA.foo__production--us-east-1.json",
  "config.live.LOLZILLA.foo__development--us-east-1.json",
  "config.live.LOLZILLA.foo__testing--us-east-1.json",
  "config.live.LOLZILLA.foo__production--sa-east-1.json",
  "config.live.LOLZILLA.foo__development--sa-east-1.json",
  "config.live.LOLZILLA.foo__testing--sa-east-1.json",
];

const createPaths = (variables, templateName) => {
  // This is the magic! Everything else is pretty trivial.
  const cartesianProduct = (...a) =>
    a.reduce((a, b) => a.flatMap((d) => b.map((e) => [d, e].flat())));

  const templateVars = [...templateName.matchAll(/\{\{(\w+)\}\}/g)].map(
    (m) => m[1]
  );

  let order = [];
  let varData = [];

  for (v of templateVars) {
    order.push(v);

    if (Array.isArray(variables[v])) {
      varData.push(variables[v]);
    } else if (typeof variables[v] === "string") {
      varData.push([variables[v]]);
    } else {
      throw new Error(
        `Invalid type in variables. Must be a string or list of strings.`
      );
    }
  }

  if (varData.length === 0) {
    return [templateName];
  }

  return cartesianProduct(...varData)
    .map((c) => {
      let _ = [];

      order.map((o, idx) => {
        _.push([o, varData.length === 1 ? c : c[idx]]);
      });

      return _;
    })
    .map((c) => {
      let _ = templateName;

      c.map(([variable, value]) => {
        _ = _.replace(`{{${variable}}}`, value);
      });

      return _;
    });
};

(() => {
  // Some other cases...s
  [
    templateName,
    `config.{{environments}}-{{regions}}-{{appName}}---{{environments}}.json`,
    `config.{{environments}}--{{environments}}.json`,
    `config.{{environments}}.json`,
    `config.json`,
  ].map((_) => {
    console.log(`Template: ${_}`);
    console.log(createPaths(variables, _));
    console.log();
  });
})();
```
