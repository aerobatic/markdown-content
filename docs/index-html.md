# Index page
In a single page application you have one entry page that is loaded when a user arrives at your application in their browser. This page is generally just a simple shim responsible for declaring static metadata like the page title and referencing the JavaScript and CSS where the interesting code lives. It may contain some of the basic page layout HTML or may not have any visible markup at all. Every Aerobatic app must have a file called `index.html` or `index.jade` that  that lives in the root of the app's source directory. Aerobatic looks for the index page in the following locations (relative to the repo root) in sequence:

1. `/app/index.(html|jade)`
2. `/src/index.(html|jade)`
3. `/index.(html|jade)`

For jade pages, the Aerobatic local development server will automatically compile to HTML avoiding the need to setup a watch and compile step in your build script.  You can read more about this in the [development server](/docs/develpement-server) docs.

## Custom Attributes
Aerobatic provides some custom attributes you can declare in your HTML to streamline and automate some common practices. 

### data-aero-build
Used to represent blocks of markup that should only be rendered in either `debug` or `release` builds. This is most commonly used to declare which JavaScript and CSS assets should be referenced. In `release` mode you will generally want to serve a small number of conactenated and minified set of assets, but in `debug` mode you want the full original source code. In the example below, only `app.min.css` and `app.min.js` will be rendered in the index page response in the release build, but omitted in `debug` mode.

```html
<!DOCTYPE html>
<html>
  <head>
  	<base href="/">
    <title>Aerobatic Starter Template</title>

    <!--
    Serve the original CSS in debug builds and a consolidated
    minfied one for release builds.
    -->
    <link data-aero-build="release" rel="stylesheet" href="/app.min.css">
    <link data-aero-build="debug" rel="stylesheet" href="/css/main.css">

    <link rel="icon" type="image/png" sizes="64x64" href="/favicon.png">
  </head>
  <body>
    <div id="centered">
      <img class="logo" src="images/logo.png">
      <h1>Aerobatic Starter Template</h1>
    </div>

    <!--
    For release builds the grunt file combines and
    minifies all scripts into a single app.min.js
    -->
    <script data-aero-build="release" src="/app.min.js"></script>

    <!-- Individual scripts are only rendered in debug mode -->
    <div data-aero-build="debug">
      <script src="/js/main.js"></script>
      <script src="/js/main.js"></script>
    </div>
  </body>
</html>
```

Although the asset URLs are coded as relative, they will be translated to absolute URLs - pointing to the CDN in production, and to localhost in [simulator mode](/docs/simulator-mode). Were you to view the source of the example app from above in production, it would look something like so:

```html
<!DOCTYPE html>
<html>
  <head>
  	<base href="/">
    <title>Aerobatic Starter Template</title>
    <link rel="stylesheet" href="//d2q4nobwyhnvov.cloudfront.net/[appId]/[versionKey]/app.min.css">
    <link rel="icon" type="image/png" sizes="64x64" href="//d2q4nobwyhnvov.cloudfront.net/[appId]/[versionKey]/favicon.png">
  </head>
  <body>
    <div id="centered">
      <img class="logo" src="images/logo.png">
      <h1>Aerobatic Starter Template</h1>
    </div>
    <script src="//d2q4nobwyhnvov.cloudfront.net/[appId]/[versionKey]/app.min.js"></script>
  </body>
</html>
```

## `__config__` global variable
If you do a view-source of an Aerobatic app you'll see a snippet right above the `<head/>` like so:

```html
<script type="text/javascript">var __config__={<json_structure>};</script>
```
Aerobatic injects this script block into `index.html` on the fly when the page is rendered. The `__config__` variable holds a JSON data structure with a bunch of contextual information about the current request that you can access from your own JavaScript. 

```js
var __config__={
  "appId":"<unique_app_id>",
  "appName":"<app_name>",
  "settings":{'setting1': 'A', 'setting2': 'B'},
  "versionId":"<current_version_id>",
  "versionName":"<current_version_name>",
  "versionKey":"21e3c9c90",
  "cdnHost":"d2q4nobwyhnvov.cloudfront.net",
  "cdnUrl":"//d2q4nobwyhnvov.cloudfront.net/<version_id>/21e3c9c90"
}
```

In `__config__` variable will also indicate whether the app is currently in simulator mode.

```js
var __config__={
  "appId":"<unique_app_id>",
  "appName":"<app_name>",
  "settings":{'setting1': 'A', 'setting2': 'B'},
  "versionId":"simulator",
  "versionName":"simulator",
  "cdnHost":"localhost:3000",
  "cdnUrl":"http://localhost:3000",
  "simulator": true,
  "simulatorPort": 3000
}
```

If you want to prevent calls to Google Analytics in simulator mode you could use code like this:
```javascript
if (__config__.simulator !== true)
  ga('create', __config__.env.GOOGLE_ANALYTICS_TRACK_CODE, {});
```

Another common scenario is loading assets asynchronously with AJAX rather than statically in the index page. In this case you can dynamically build the absolute URL with the `__config__.cdnUrl`.

```js
$.ajax({
  url: __config__.cdnUrl + '/templates/template.html'
});
```

Of course if you have a healthy aversion to global variables you can always abstract `__config__` away somewhere. In an Angular app the [angular-aerobatic](https://www.npmjs.org/package/angular-aerobatic) module wraps the `__config__`object in a `aerobatic` dependency that you can inject into your modules. The [angular-seed template](https://github.com/aerobatic/angular-seed) comes with this pre-configured for you.

The name `__config__` was modeled after Python's 'dunder' convention for magic methods and variables. Plus the double underscores kinda look like little wings;)
