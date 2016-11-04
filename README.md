# Ionic 2 Progressive Web App with Webpack Demo
> Angular Hack Day Sydney Nov 2016

This repository contains the Ionic 2 Progressive Web App Demo presented at the [Angular Hack Day Sydney](http://angularhackday.com/sydney/) in November 2016. The slides of the presentation are available [here](https://slides.com/julienevano/native-and-progressive-web-app-with-ionic2-and-webpack):
<iframe src="http://slides.com/julienevano/native-and-progressive-web-app-with-ionic2-and-webpack/embed" width="576" height="420" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

## Tutorial
This tutorial is using:
* Node.js 5.x and higher
* NPM 3.x as a package manager
* Ionic 2.0.0-rc1
* Ionic App Scripts 0.0.37-5
* Webpack 2
* TypeScript 2

> Due to continuous evolution of the Ionic and Ionic App Scripts project, this tutorial can become obsolete in future versions, but it can still give an idea of how to build an Ionic 2 Progressive Web App with Webpack.

The tutorial is following severals steps that you can walk through yourself and build the application from the ground up. However, if you get stuck or want to start from a clean slate, each step has an associated Git tag that you can checkout to reset your code to match the start of the associated step in the tutorial. For some of the steps, an install of the dependencies is also needed.

The available step tags in order are:
* `step-1`
* `step-2` (needs to run `npm install`)
* `step-3` (needs to run `npm install`)
* `step-4`
* `step-5` (needs to run `npm install`)

For instance, to start on Step 2, run `git checkout step-2 && npm install`.

### Prerequirements
This tutorial assumes that you have already installed Node.js and NPM on your machine, cloned the repository and executed the following commands:
* `npm install -g ionic`

## Steps
### Initial step - Create and build an Ionic 2 project
In this step, we have already created a new Ionic 2 project with a default template (tabs layout) using `ionic start angularhackday-ionic2-pwa-demo --v2`. If you want to create a new project by following those steps, you can re-run this command with the name of your project; otherwise please run `git checkout step-1`.

### Step 1 - Install Ionic App Scripts Beta
By default, Ionic App Scripts is using Rollup.js to build the project. In this step, we are installing the beta version in order to use Webpack.
```bash
# Install the beta version of Ionic App Scripts
npm install @ionic/app-scripts@beta --save-dev
```

### Step 2 - Extend Ionic App Scripts copy and webpack task configs
#### Create the config files
* Create a new `config` folder at the root of the project and two empty files `copy.config.js` and `webpack.config.js`:
```
.
+-- config
|   +-- copy.config.js
|   +-- webpack.config.js
```
* Copy the content of `node_modules/@ionic/app-scripts/config/copy.config.js` into the new `copy.config.js` file, and add the following copy (because it is required by the manifest):
```
{
  src: 'resources/icon.png',
  dest: '{{WWW}}/assets/imgs/logo.png'
}
```
* In `webpack.config.js`, merge the config of `@ionic/app-scripts/config/webpack.config` with an empty config for now:
```javascript
var ionicWebpackConfig = require('@ionic/app-scripts/config/webpack.config');
var webpackMerge = require('webpack-merge');

// Webpack config
module.exports = webpackMerge(ionicWebpackConfig, {
});
```
* Install the `webpack-merge` dependency:
```bash
npm install webpack-merge --save-dev
```

#### Register the custom config files
Within the `package.json` file, add the following `config` property in order to reference our new custom config files. We also set the build directory to be at the root of `www` instead of a subfolder `www/build` (it will be used later by the Webpack offline plugin to generate the service worker at the root of the web app).
```json
"config": {
  "ionic_copy": "./config/copy.config.js",
  "ionic_webpack": "./config/webpack.config.js",
  "ionic_build_dir": "www"
},
```

### Step 3 - Update the Manifest
By default, the generated manifest is quite simple and doesn't reflect the name and description of your app. In this step, we are updating and completing the `manifest.json` file:
```json
{
  "name": "Angular Hack Day Ionic 2 PWA",
  "short_name": "AngularHackDay",
  "description": "Ionic 2 PWA demo for Angular Hack Day - Sydney",
  "start_url": "index.html",
  "display": "standalone",
  "orientation": "portrait",
  "icons": [{
    "src": "assets/imgs/logo.png",
    "sizes": "512x512",
    "type": "image/png"
  }],
  "background_color": "#4e8ef7",
  "theme_color": "#4e8ef7"
}
```

### Step 4 - Extend Webpack config
By default, the manifest and the service worker generated by Ionic are quite simple and don't allow you to customise them. In this step, we will see how to customise our Webpack config in order to automatically generate:
* Over 30 favicons (configurable) for Android, iOS, and the different desktop browsers from one source png.
* A solid base html page with all your webpack generated css and js files built in.
* Service worker and app cache as a fallback for the best offline experience of a Progressive Web App.

#### Install the Webpack plugins
```bash
npm install favicons-webpack-plugin --save-dev

npm install html-webpack-plugin --save-dev

npm install offline-plugin --save-dev
```

#### Add the Webpack plugin configs
First we have to add a new `plugins` property in our empty Webpack config:
```javascript
module.exports = webpackMerge(ionicWebpackConfig, {
  plugins: []
});
```
##### HtmlWebpackPlugin config
This plugin is in charge of adding all css, js and image files to the `index.html` template.
```javascript
var HtmlWebpackPlugin = require('html-webpack-plugin');

...

module.exports = webpackMerge(ionicWebpackConfig, {
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html',
      filename: 'index.html',
      chunksSortMode: 'dependency',
      options: {
        title: manifest.name
      }
    })
  ]
});
```

##### FaviconsWebpackPlugin config
This plugin is in charge of generating the icons for Android, iOS, and the different desktop browsers from the icon defined in our Ionic project. In addition, when used with the HtmlWebpackPlugin, it also adds the references to those images and manifest information into the generated `index.html`.
```javascript
var manifest = require('../src/manifest.json');
var FaviconsWebpackPlugin = require('favicons-webpack-plugin');

...

module.exports = webpackMerge(ionicWebpackConfig, {
  plugins: [
    new HtmlWebpackPlugin(...),

    new FaviconsWebpackPlugin({
      logo: path.resolve(__dirname, '../resources/icon.png'),
      prefix: 'assets/icons/favicons-[hash]/',
      background: manifest.background_color,
      title: manifest.name,
      icons: {
        android: true,
        appleIcon: true,
        appleStartup: true,
        coast: false,
        favicons: true,
        firefox: true,
        opengraph: false,
        twitter: true,
        yandex: false,
        windows: false
      }
    })
  ]
});
```

##### OfflinePlugin config
This plugin is in charge of generating a service worker and an app cache managing the cache and the update of all our files.
```javascript
var package = require('../package.json');
var path = require('path');
var OfflinePlugin = require('offline-plugin');

...

module.exports = webpackMerge(ionicWebpackConfig, {
  plugins: [
    new HtmlWebpackPlugin(...),
    new FaviconsWebpackPlugin(...),

    new OfflinePlugin({
      caches: {
        main: [
          'index.html',
          'main.css',
          'polyfills.js',
          'main.js',
          'assets/icon/favicon.ico',
          'assets/icon/favicons-*/favicon.ico',
          'assets/fonts/ionicons.woff2',
          'assets/fonts/ionicons.woff',
          'assets/fonts/ionicons.ttf',
          'assets/imgs/logo.png'
        ],
        additional: [
          'manifest.json',
          'assets/icons/favicons-*/*.png'
        ],
        optional: [
        ]
      },
      externals: [
        'polyfills.js',
        'assets/fonts/*.*',
        'assets/imgs/logo.png'
      ],
      excludes: ['**/*.gz', '**/.cache'],
      updateStrategy: 'all',
      version: package.version + '.[hash]',

      relativePaths: true,

      ServiceWorker: {
        output: 'sw.js'
      },

      AppCache: {
        directory: 'appcache/'
      }
    })
  ]
});
```
> We are using the npm package version to determine the version of the service worker. In order for it to work, please add a version property into the `package.json` file.

#### Update the `index.html` file
* Clean up the `index.html` template:
```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
  <meta charset="UTF-8">
  <title><%= htmlWebpackPlugin.options.title %></title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <meta name="format-detection" content="telephone=no">
  <meta name="msapplication-tap-highlight" content="no">

  <link rel="manifest" href="manifest.json">

  <!-- cordova.js required for cordova apps -->
  <!--<script src="cordova.js"></script>-->

  <link href="main.css" rel="stylesheet">
</head>
<body>
  <!-- Ionic's root component and where the app will load -->
  <ion-app></ion-app>

  <!-- The polyfills js is generated during the build process -->
  <script src="polyfills.js"></script>
</body>
</html>
```
* Remove the copy of the index.html as it is now done by `HtmlWebpackPlugin`:
```json
{
  src: '{{SRC}}/index.html',
  dest: '{{WWW}}/index.html'
}
```

#### Replace the current service worker with an service worker install
We remove the service worker generated by Ionic and replace it with the one generated by the Wepack Offline plugin:
* Delete the file `src/service-worker.js`.
* Remove the copy of the index.html and the service worker in the copy task config:
```json
{
  src: '{{SRC}}/service-worker.js',
  dest: '{{WWW}}/service-worker.js'
},
```
* Create a new file `src/service-worker-install.ts` with the following script:
```javascript
(<{install: Function}>require('offline-plugin/runtime')).install();
```
* Import the service worker install in `main.dev.ts` and `main.prod.ts`:
```
require('../service-worker-install');
```
* Add typings for `require` into `declarations.d.ts`:
```
declare var require: {
  <T>(path: string): T;
  (paths: string[], callback: (...modules: any[]) => void): void;
  ensure: (paths: string[], callback: (require: <T>(path: string) => T) => void) => void;
};
```

#### Add a platform condition before setup cordova plugins
By default Ionic is always setting up the status bar and the splashcreen at start in `src/app/app.component.ts`. However this is only valid if the app is running on a device. We will add a condition to run it only on Cordova:
```javascript
platform.ready().then(() => {
  if (platform.is('cordova')) {
    // Okay, so the platform is ready and our plugins are available.
    // Here you can do any higher level native things you might need.
    StatusBar.styleDefault();
    Splashscreen.hide();
  }
});
```

### Step 5 - Finished
Now we have setup our Progressive Web App with Ionic 2 and Wepack, we can run it:
```bash
ionic serve
```

## Summary
With only few steps and a small amount of work, we have quickly created a functional Progressive Web App with Ionic 2 and Webpack which automatically generates resources, a service worker and an app cache as fallback for incompatible browsers.