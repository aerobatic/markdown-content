## The "index.html" file
In a single page application you have one entry page that is loaded when a user arrives at your application in their browser. This page is generally just a simple shim responsible for declaring static metadata like the page title and referencing the JavaScript and CSS that contain where the interesting code lives. It may contain some of the basic page layout HTML or it may not have any visible markup at all. Every Aerobatic app must have a file called `index.html` that lives in the root of the app (which could be built from a Grunt task if you want to use a syntax like [Jade](http://jade-lang.com) or [Haml](http://haml.info)).

```html
<html>
  <head>
    <title>Aerobatic Starter Template</title>

    <!--
    Serve the original CSS in debug builds and a consolidated
    minfied one for release builds.
    -->
    <link data-aero-build="release" rel="Stylesheet" href="dist/app.min.css">
    <link data-aero-build="debug" rel="Stylesheet" href="css/main.css">

    <link rel="icon" type="image/png" sizes="64x64" href="favicon.png">
  </head>
  <body>
    <div id="centered">
      <img class="logo" src="images/logo.png">
      <h1>Aerobatic Starter Template</h1>
    </div>

    <!--
    Here you might declare 3rd party library scripts. You could point
    to a public CDN or to local copies, perhaps managed through bower.
    -->

    <!--
    For release builds the grunt file combines and
    minifies all scripts into a single app.min.js
    -->
    <script data-aero-build="release" src="dist/app.min.js"></script>

    <!-- Individual scripts are only rendered in debug mode -->
    <div data-aero-build="debug">
      <script src="js/main.js"></script>
      <script src="js/main.js"></script>
    </div>
  </body>
</html>
```

Everything here looks pretty standard. One of the goals of Aerobatic is to make app development as familiar as possible. However you might have noticed the `data-aero-build` attribute. While Aerobatic apps consist almost entirely of static assets, the index.html document is special in that it undergoes some lightweight server processing, including stripping out blocks that do not match the current build type. So for example if this document was being served in production, the `data-aero-build="debug"` elements would get stripped out and conversely the `release` elements are removed in simulator mode (more about that soon).

A typical best practice is to serve just a single consolidated JavaScript and Stylesheet in production mode. But during development you want to keep your scripts separate for easier debugging. Aerobatic makes it really easy to configure this without a special build step.

Another part of the index.html processing entails converting asset URLs to absolute URLs to point to a CDN or the local simulator as appropriate.

### `__config__` global variable
If you do a view-source of an Aerobatic app you'll see a snippet right above the `<head/>` like so:

```html
<script type="text/javascript">var __config__={<json_structure>};</script>
```
Aerobatic injects this script block into `index.html` on the fly when the page is rendered. The `__config__` variable holds a JSON data structure with a bunch of contextual information about the current request. Since it is global to the page any of your JavaScript can access it. One common scenario is to include the versionName in calls to your web analytics so you can see reports segmented by app version (something you'll definitely want to consider in conjunction with split testing using [Traffic Control](/docs/traffic-control) rules). You can also dynamically build paths to assets in client code with `__config__.cdnUrl`.

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

If you, for example, want to prevent calls to Google Analytics in simulator mode you could use code like this:
```javascript
if (__config__.simulator !== true)
  ga('create', __config__.settings.GOOGLE_ANALYTICS_TRACK_CODE, {});
```

Of course if you have a healthy aversion to global variables you can always abstract `__config__` away somewhere. In an angular app you might wrap it in a service that can be injected wherever it is needed.

```js
angular.module('my-app').value('aerobaticConfig', window.__config__);
```

The name `__config__` was modeled after Python's 'dunder' convention for magic methods and variables. Plus the double underscores kinda look like little wings;)

