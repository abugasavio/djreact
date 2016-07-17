## Django React

This project is about making two technologies that I love - Django and React - play together.

### Adding django-webpack-loader

First of all you need to run `pip install django-webpack-loader` and add it to `requirements.txt`.

Next you need to add this reusable Django app to the `INSTALLED_APP` setting in the settings.py:

```python
INSTALLED_APPS = [
    ...
    'webpack_loader',
]

Next we need to create a `package.json` file:


```json
{
  "name": "djreact",
  "version": "0.0.1",
  "devDependencies": {
    "babel": "^6.5.2",
    "babel-core": "^6.6.5",
    "babel-eslint": "^5.0.0",
    "babel-loader": "^6.2.4",
    "babel-plugin-transform-decorators-legacy": "^1.3.4",
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-react": "^6.5.0",
    "babel-preset-stage-0": "^6.5.0",
    "eslint": "^2.2.0",
    "react": "^0.14.7",
    "react-hot-loader": "^1.3.0",
    "redux-devtools": "^3.1.1",
    "webpack": "^1.12.13",
    "webpack-bundle-tracker": "0.0.93",
    "webpack-dev-server": "^1.14.1"
  },
  "dependencies": {
    "es6-promise": "^3.1.2",
    "isomorphic-fetch": "^2.2.1",
    "lodash": "^4.5.1",
    "radium": "^0.16.6",
    "react-cookie": "^0.4.5",
    "react-dom": "^0.14.7",
    "react-redux": "^4.4.0",
    "redux": "^3.3.1",
    "redux-thunk": "^1.0.3"
  }
}
```

One of the packages that will be installed is [Babel](http://babeljs.io) which is a tool that "transpiles" ES6 to a
Javascript syntax that browsers support.

When the `package.json` file has been created, you can install the packages via `npm install`.
This will create a `node_modules` folder, so we should add that folder to `.gitignore`. 

### Using Webpack

We need to create some config files.
1. `webpack.base.config.js`

```javascript
var path = require("path")
var webpack = require('webpack')

module.exports = {
  context: __dirname,

  entry: {
    // Add as many entry points as you have container-react-components here
    App1: './reactjs/App1',
    vendors: ['react'],
  },

  output: {
      path: path.resolve('./djreact/static/bundles/local/'),
      filename: "[name]-[hash].js"
  },

  externals: [
  ], // add all vendor libs

  plugins: [
    new webpack.optimize.CommonsChunkPlugin('vendors', 'vendors.js'),
  ], // add all common plugins here

  module: {
    loaders: [] // add all common loaders here
  },

  resolve: {
    modulesDirectories: ['node_modules', 'bower_components'],
    extensions: ['', '.js', '.jsx']
  },
}
```

Here is what it does:

* It defines the entry point. That is the JS-file that should be loaded first.
* It defines the output path. That is where you want to save our bundle
* It used the `CommonsChunksPlugin`, this makes sure that ReactJS will be saved as a different files(`vendors.js`), so
  that our actual app-bundle doesn't become too big.
  
2. 