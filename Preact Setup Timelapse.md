Notes from setting up [a Preact-based SPA](https://github.com/afreeorange/preact-boilerplate-timelapse) from ground up.

### Initial Setup

```bash
# Initialize the project
npm init

# Install webpack stuff
npm install webpack@3.11.0 webpack-cli@2.0.9 webpack-dev-server@3.0.0 --save-dev
```

I had to install those specific versions. Webpack 4-beta didn't work for me. `webpack-dev-server@3.1.0` game me this delightful error

```
/Users/nikhil/Downloads/git-preact-wiki/node_modules/webpack-dev-server/bin/webpack-dev-server.js:405
    throw e;
    ^

TypeError: Cannot read property 'plugin' of undefined
```

The `stylelint-loader` didn't work with Webpack 4 and `webpack-dev-server`. Frontend development is _exciting_.

My project [looked like this](https://github.com/afreeorange/preact-boilerplate-timelapse/tree/1f6bf9bd17279b187f0805ae0657dfa89b7ed651) with a [base Webpack config](https://github.com/afreeorange/preact-boilerplate-timelapse/blob/1f6bf9bd17279b187f0805ae0657dfa89b7ed651/webpack.config.js).

I then created a [source folder](https://github.com/afreeorange/preact-boilerplate-timelapse/tree/eee2b920251ec08c9fa04353af4dfc869b037d09) with a simple [App.js](https://github.com/afreeorange/preact-boilerplate-timelapse/blob/eee2b920251ec08c9fa04353af4dfc869b037d09/src/App.js).

Added [two commands](https://github.com/afreeorange/preact-boilerplate-timelapse/blob/1f6bf9bd17279b187f0805ae0657dfa89b7ed651/package.json#L7) to `package.json`: one that builds the project and another that launches the live-reloading dev server.

### Babel


```bash
npm i -D babel-loader @babel/core @babel/preset-env
```

I [updated the Webpack config](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/ac3acccd0a68b9092c302bf1c2f15028c491d95e) to use the loader and tested.

### HTML Template

```bash
npm i -D html-webpack-plugin
```

Created a small template for the SPA called [`./src/index.html`](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/323fd386807c374096edd7e38a680c25161752f2#diff-e249faefed5757034596c5096d33dab6) and [updated the Webpack config](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/323fd386807c374096edd7e38a680c25161752f2#diff-11e9f7f953edc64ba14b0cc350ae7b9d) to use it.

At this point, running `npm start` successfully showed me my SPA.

### Preact and a simple SPA

```bash
npm i -D preact
```

[Updated `src/App.js`](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/36e1613fe31d33e6b3864364c284d05159b48338#diff-14b1e33d5bf5649597cdc0e4f684dadd) to import Preact and display a simple component.

### JSX

```bash
npm i -D babel-plugin-transform-react-jsx
```

[Updated the Webpack config](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/f8e2435abbc602118320931e912d31b88be337a9#diff-11e9f7f953edc64ba14b0cc350ae7b9d) to use the plugin and tried [using JSX in `src/App.js`](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/f8e2435abbc602118320931e912d31b88be337a9#diff-14b1e33d5bf5649597cdc0e4f684dadd). So far so good.

### Linting ES6

```bash
# Linting. Using the AirBnB guide. Make sure ESLint doesn't barf on JSX
npm i -D eslint \
         eslint-loader \
         eslint-plugin-import \
         eslint-config-airbnb-base \
         eslint-plugin-react \
         babel-eslint
```

Added a [basic ESLint configuration](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/fad85772c0ee30d7db1552c71d0e0b3611a5d9d4#diff-df39304d828831c44a2b9f38cd45289c) to the root of the project and [updated Webpack](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/fad85772c0ee30d7db1552c71d0e0b3611a5d9d4#diff-11e9f7f953edc64ba14b0cc350ae7b9d) to use the loader. Now I have linting with the live-reload server :)

### Clean Before Build

```bash
npm i -D clean-webpack-plugin
```

[Updated Webpack](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/71b0ccdb30ad834aeeb1a6b379807ef8d814243c#diff-11e9f7f953edc64ba14b0cc350ae7b9d) to use the plugin. Easy-peasy.

### Uglify JS Output

```
npm i -D uglifyjs-webpack-plugin
```

And [updated Webpack](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/467c225358d225086514891b5c3827010943fbe5#diff-11e9f7f953edc64ba14b0cc350ae7b9d).

### Getting Sassy

Wanted to add Sass support and create a separate, compiled, minified file `App.css` in the output

```bash
npm i -D extract-text-webpack-plugin style-loader css-loader sass-loader node-sass
```

Added [a simple Sass file](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/c1189057ca0c36d035f8d4f5e5d0f7643e6da0c8#diff-f03d0bbe56a074dd841d993fa82c1bfb) and [updated Webpack](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/c1189057ca0c36d035f8d4f5e5d0f7643e6da0c8#diff-11e9f7f953edc64ba14b0cc350ae7b9d) to use the plugins and spit out `App.css`.

### Add Sass Linting

```bash
# Does not work with Webpack 4 beta and the dev server
npm i -D stylelint stylelint-webpack-plugin stylelint-config-standard
```

Added [a basic StyleLint config](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/e3f4312bd59452a3b3c162557ed6b1fb52f313b5#diff-3d9639f99b2cd6bda90b892bea090bef) and [updated Webpack](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/e3f4312bd59452a3b3c162557ed6b1fb52f313b5#diff-11e9f7f953edc64ba14b0cc350ae7b9d) to use it.

### Assets &rarr; Base64

```bash
# Does not work with Webpack 4 beta
npm i -D base64-inline-loader
```

[Updated Webpack](https://github.com/afreeorange/preact-boilerplate-timelapse/commit/966c1b406968ccc60a173d55db84a4fc1f4c15cb#diff-11e9f7f953edc64ba14b0cc350ae7b9d)
