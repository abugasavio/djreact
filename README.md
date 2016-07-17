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
  
2. `webpack.local.config.js`

```javascript
var path = require("path")
var webpack = require('webpack')
var BundleTracker = require('webpack-bundle-tracker')
var config = require('./webpack.base.config.js')

config.devtool = "#eval-source-map"

config.plugins = config.plugins.concat([
  new BundleTracker({filename: './webpack-stats-local.json'}),
])

config.module.loaders.push(
  { test: /\.jsx?$/, exclude: /node_modules/, loaders: ['react-hot', 'babel'] }
)

module.exports = config
```

This loads the vase config and then adds a few things to it, most notably one more plugin: The `BundleTracker` plugin.

This plugin creates a JSON file every time we generate bundles. Django can then read that JSON file and will know which
bundle belongs to which App-name.

Since we will be using ES2015 Javascript syntax for all our JavaScript code we need `babel` to "transpile" the advanced
code back to something that browsers can understand. For this to work we need `.babelrc` file:imp

```json
{
  "presets": ["es2015", "react", "stage-0"],
  "plugins": [
    ["transform-decorators-legacy"],
  ]
}
```

### Writing reactjs code

We create a `reactjs` folder and put a `App1.jsx` file inside. This is going to be one of our entry point for bundling.
`webpack` will look into that file and then follow all its imports and add the to the bundle, so at the end we will have one big
`App1.jsx` file that can be used in the browser.

```javascript
import React from "react"
import { render } from "react-dom"

import App1Container from "../containers/App1Container"

class App1 extends React.Component {
    render() {
        return (
            <App1Container />
        )
    }
}

render(<App1/>, document.getElementById('App1'))
```

The above file tries to import another component called `App1Container`. So we create it in `containers/AppContainer.jsx`.

```javascript
import React from "react"

import Headline from "../components/Headline"

export default class App1Container extends React.Component {
  render() {
    return (
      <div className="container">
        <div className="row">
          <div className="col-sm-12">
            <Headline>Sample App!</Headline>
          </div>
        </div>
      </div>
    )
  }
}
```

And once again, that component imports another component called `Headline`.
Let's create that one as well in `components/Headline.jsx`:

```javascript
import React from "react"

export default class Headline extends React.Component {
  render() {
    return (
      <h1>{ this.props.children }</h1>
    )
  }
}
```

You might wonder why I am using a component `App1` and another one
`App1Container`. This will make more sense a bit later. We will be using
something called `Redux` to manage our app's state and you will see that Redux
requires quite a lot of boilerplate to be wrapped around your app. To keep my
files cleaner, I like to have one "boilerplate" file, which then imports the
actual ReactJS component that I want to build.

You will also notice that I separate my components into a `containers` folder
and into a `components` folder. You can think about this a bit like Django
views. The main view template is your container. It contains the general
structure and markup for your page. In the `components` we will have much
smaller components that do one thing and one thing well. These components will
be re-used and orchestrated by all our `container` components, they would be the
equivalent of smaller partial templates that you import in Django using the
`{% import %}` tag.

At this point you can run `node_modules/.bin/webpack --config webpack.local.config.js`
and it should generate some files in `djreact/static/bundles/`.

### Using the bundle

Upto now we have been able to create a bundle but have not been able to see the result in a browser.

To be able to do this we need to change the templates.

Update `view1.html` so that it looks like this:

```html
{% extends "base.html" }
{% load render_bundle from webpack_loader %}

{% block main %}
<div id="App1"></div>
{% render_bundle 'vendors' %}
{% render_bundle 'App1' %}
{% endblock %}
```

We also need to add a new setting to `settings.py`:

```python
WEBPACK_LOADER = {
    'DEFAULT': {
        'BUNDLE_DIR_NAME': 'bundles/local/',  # end with slash
        'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats-local.json'),
    }
}
```

`BUNDLE_DIR_NAME` tells Django in which folder within the `static` folder it can find the bundles

`STATS_FILE` tells django where it can find the JSON-file that maps entry-point names to bundle files. It is because of
this stats file that we can use `{% render_bundle 'App1' %}` in our template.  You will also find this `App1` name in 
your `webpack.base.config.js` file under the entry attribute.

Now run `./manage.py runserver` and visit your site. You should see `"Sample App!"`.

Now try to make a change to your ReactJS app. Change `Sample App!` to `Something New!` in `containers/App1Container.jsx`.

Then run node_modules/.bin/webpack --config webpack.local.config.js again, make sure that `./manage.py runserver` is still running and visit your site in the browser. It should say `"Something New!"` now.

### Hot reloading

Here we wan to create local bundles without re-running the `webpack` command every time we change our ReactJS app.

First, we need a `server.js` file that will start a webpack-dev-server for us:

```javascript
var webpack = require('webpack')
var WebpackDevServer = require('webpack-dev-server')
var config = require('./webpack.local.config')

new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  inline: true,
  historyApiFallback: true,
}).listen(3000, config.ip, function (err, result) {
  if (err) {
    console.log(err)
  }

  console.log('Listening at ' + config.ip + ':3000')
})
```

Next, we need to add/replace the following in our `webpack.local.config.js`:

```javascript
var ip = 'localhost'

config.entry = {
  App1: [
    'webpack-dev-server/client?http://' + ip + ':3000',
    'webpack/hot/only-dev-server',
    './reactjs/App1',
  ],
}

config.output.publicPath = 'http://' + ip + ':3000' + '/assets/bundles/'

config.plugins = config.plugins.concat([
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NoErrorsPlugin(),
  new BundleTracker({filename: './webpack-stats-local.json'}),
])
```

Ready? In one terminal window, start the webpack-dev-server with node server.js and in another terminal window, 
start the Django devserver with `./manage.py runserver`.

Make sure that you can still see "Something New!".

And now change it to Something Fancy! in `containers/App1Container.jsx` and switch back to your browser. 
If you are very fast, you can see how it updates itself.

There is another cool thing: When you open the site in Google Chrome and 
open the developer tools with COMMAND+OPTION+i and then open the Sources tab, 
you can see webpack:// in the sidebar. It has a folder called . where you will find the original ReactJS sources. 
You can even put breakpoints here and debug your app like a pro. 
No more console.log() in your JavaScript code.

