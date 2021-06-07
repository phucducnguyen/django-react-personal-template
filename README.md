<div align="center">
  <h1><code>Django-React</code></h1>

  <!-- <strong>Personal template</strong> -->

  <sub>Personal Template</sub>
</div>

### this is basic set up template to use React with Django - (django backedn - react frontend)

---
---
## Install Django & Setup Back-end
*create folder*
```bash
$mkdir django-react
$cd django-react
```
*install django*
```bash
$pip install pipenv
$pipenv shell
$pipenv install django
```
## Back-end (Django)
```bash
$django-admin startproject backend
$cd backend
$python manage.py migrate #python3 if use ubuntu
```

## Front-end (React)
```bash
$python manage.py startapp frontend
```
<br/>
---

*Add `frontend` in the project's `settings.py` file*
```python
INSTALLED_APPS = [
  'frontend',
  ...
]
```
*Create folder `frontend/templates/frontend/` then create `index.html`*
```html
<!-- create frontend/templates/frontend/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Site</title>
</head>
<body>
  <div id="app"></div> <!-- this is where REACT app is rendered -->
</body>
</html>
```

*In `frontend/views.py` add*
```python
from django.shortcuts import render

# this function will render index.html 
def index(request):
  return render(request, 'frontend/index.html')
```

*Edit `urlpatterns` in `backend/urls.py` to tell **Django** the url which it find index.html*
```python
from django.urls import include, path

urlpatterns = [
  ...,
  path('', include('frontend.urls'))
]
```
*In `frontend` folder, create a urls.py file and put the following in it.*
```python
from django.urls import path
from . import views

urlpatterns = [
  path('', views.index)
]
```
---
---
<br/>

## Set Up React, Babel, and Webpack
*Inside folder `frontend/` run:
```bash
#Insde frontend/
$npm init -y
$npm install react react-dom
$npm install --save-dev @babel/core
$npm install --save-dev @babel/preset-env @babel/preset-react
$npm install --save-dev webpack webpack-cli webpack-bundle-tracker@1.0.0 babel-loader css-loader style-loader clean-webpack-plugin
```
*That's quite a few! Let's break it down:*
- `Babel` is a JavaScript transpiler, which essentially means it lets you use things in your JavaScript code (like JSX) that the browser wouldn't understand natively.
- By default, Babel does nothing. If you want Babel to transpile a specific thing in your JavaScript code, you need to install a plugin for it. Your project might need several plugins, so Babel also has this concept of presets, which are just collections of plugins. 
- `@babel/preset-env` is a collection of plugins that allows you to use the latest JavaScript features even if your browser doesn't support them yet.
- `@babel/preset-react` is a collection of plugins that allows you to do React things in a nice way, like use JSX instead of nested calls to `React.createElement`.

- `webpack` is... well, Webpack
- `webpack-cli` allows to run Webpack commands from the command line
- `webpack-bundle-tracker` is a plugin that writes some stats about the bundle(s) to a JSON file.
- `babel-loader` is a loader that tells Webpack to run Babel on the file before adding it to the bundle.
- `css-loader` and `style-loader` are loaders that allow you to import .css files into your JavaScript
    clean-webpack-plugin is a plugin that deletes old bundles from Webpack's output directory every time a new bundle is created.
<br/>
<br/>

*Create `.babelrc` and `webpack.config.js` inside `frontend/` folder
```js
// frontend/.babelrc
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```
```js
// frontend/webpack.config.js
const path = require('path');
const webpack = require("webpack");
const BundleTracker = require('webpack-bundle-tracker');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    entry: {
        frontend: "./src/index.js",
    },
    output: {
        path: path.resolve(__dirname, "./static/frontend"),
        filename: "[name]-[hash].js",
        publicPath: 'static/frontend/',
    },
    module: {
        rules: [
            {
                test: /\.js|.jsx$/,
                exclude: /node_modules/,
                use: "babel-loader",
            },
            {
                test: /\.css$/,
                exclude: /node_modules/,
                use: ["style-loader", "css-loader"],
            },
        ],
    },
    optimization: {
        minimize: true,
    },
    plugins: [
        new CleanWebpackPlugin(),
        new BundleTracker({
            path: __dirname,
            filename: './webpack-stats.json',
        }),
        new webpack.DefinePlugin({
            "process.env": {
                NODE_ENV: JSON.stringify("development"),
            },
        }),
    ],
};
```

- `entry` tells Webpack where to start gathering your code
- `output` is where Webpack will put the finished bundle.
- `plugins` tells Webpack which plugins to use
- `module` is where you configure your loaders. Each rule tells Webpack that whenever it comes across a file that matches the test regex, it should use the specified loaders to process it.
<br/>
<br/>

*Add some scripts to `package.json` to run `Webpack`*
```js
// frontend/package.json
{
  ...,
  "scripts": {
    ...,
    "dev": "webpack --config webpack.config.js --watch --mode development",
    "build": "webpack --config webpack.config.js --mode production"
  }
}
```
These scripts create development bundle with `$npm run dev` and `$npm run build`
<br/>
<br/>

---
---
## Add Django bundle to HTML
To be able to add JavaScript bundle to HTML page, `django-webpack-loader` is needed:
```bash
# backend/ folder -- root directory
$pip install django-webpack-loader
```
*Go back to `settings.py` file, add:*
```python
import os   #<-- this line

...

INSTALLED_APPS = [
  'webpack_loader',  #<-- this line
  ...
]

...
# And this block
WEBPACK_LOADER = {
    'DEFAULT': {
        'BUNDLE_DIR_NAME': 'frontend/',
        'STATS_FILE': os.path.join(BASE_DIR, 'frontend/webpack-stats.json')
    }
}
```
<br/>

*Then add `template tag` to load the `bundle` into your page*
```html
<!DOCTYPE html>
{% load render_bundle from webpack_loader %} <!-- add this line -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Phuc Nguyen</title>
</head>
<body>
    <div id="app"></div>
    {% render_bundle 'frontend' %} <!-- add this line -->
</body>
</html>
```
<br/>

---
---

## Create React App
Now, all the parts are in place. Lets make React Application
* In folder `frontend/` create folder `src/`.
* In folder `src/` create `App.js` and `index.js`.

```js
// frontend/src/App.js
import React from 'react'

const App = () => {
  return (
    <div>Hello, World!</div>
  )
}

export default App

```
```js
// frontened/src/index.js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

ReactDOM.render(
  <App />,
  document.getElementById('app')
)

```
---
---
## Run
Open 2 terminals:
1. First terminal: 
```bash
$npm run device
```
2. Second terminal:
```bash
$python manage.py runserver
```
3. Navigate to `localhost:8000` in the browser

---
---
## More:
[React Django integration](https://medium.com/analytics-vidhya/django-react-integration-37acc304e984) <br/>
[How to Serve a React Single-Page App with Django](https://dev.to/zachtylr21/how-to-serve-a-react-single-page-app-with-django-1a1l) <br/>
Digital Ocean: [How To Build a To-Do application Using Django and React](https://www.digitalocean.com/community/tutorials/build-a-to-do-application-using-django-and-react)




