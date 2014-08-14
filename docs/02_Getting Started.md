Aerobatic's goal is to get you up and running in 10 minutes or less. First off you're going to need node.js. If you don't already have that, head on over to [http://nodejs.org](http://nodejs.org) and install it for your platform.

Ok, now that you've got node installed, next we need to install Grunt. You'll be using it constantly when developing on the Aerobatic platform to build your assets, run your local simulator server, deploy new versions to the cloud, execute unit tests, and more. Installation is a breeze with npm (which was installed as part of node).

```bash
npm install -g grunt-cli
```

## Building Your First App
1. Login to your Aerobatic dashboard and click the `Create App` button
2. Choose a name for your app. The URL for your app will be `http://<your-app-name>.aerobaticapp.com`
3. Follow the instructions

One of the steps is to create a dot file in the root of your app directory called `.aerobatic` which contains a small JSON structure in the form:
```js
{
  "appId": "<unique_app_id>",
  "secretKey": "<secret_key>",
  "userId": "<aerobatic_user_id>"
}
```

The `secretKey` is your individual access key for performing grunt operations like deploying new versions for this app. Since it is sensitive you should ensure that your `.gitignore` file specifically excludes this file.

Rather than starting from a blank slate, you can `git clone` one of the samples in the [app gallery](/gallery). For your first app, the [Starter Template](https://github.com/aerobatic/starter-template) is recommended (which the code in this article is taken from). The following terminal commands will clone the starter app, and use `npm` to install any grunt dev dependencies in declared in the `package.json`.

```bash
> git clone git@github.com:aerobatic/starter-template.git aerobatic-starter
> cd aerobatic-starter
> npm install && npm update grunt-aerobatic --save-dev
```

## Grunt
If you're not familiar with [Grunt](http://gruntjs.com), you should go read up on it. In a nutshell, it's a very powerful yet simple command line tool for automating most any repetitive web development task. Best of all it has a robust community driven ecosystem of plugins and is in use by many major tech companies and open-source projects. Rather than invent a new command line tool, Aerobatic has chosen to extend Grunt with custom tasks that complement the rich set of existing plugins. As new technologies emerge, Grunt plugins will quickly follow, and you can be confident that your Aerobatic apps will continue to be compatible with the latest and greatest the web has to offer.

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


## Gruntfile.js (or .coffee)
The last required file (in addition to `.aerobatic`, `package.json`, and `index.html`) is your `Gruntfile.js`. There's plenty of documentation on the Grunt site, so this will only cover the Aerobatic specific setup.

```js
module.exports = function(grunt) {
  grunt.initConfig({
    watch: {
      options: {
        spawn: true,
        livereload: true
      },
      index: {
        files: ['index.html']
      },
      css: {
        files: ['css/*.css']
	    },
      scripts: {
        files: ['js/**/*.js'],
        tasks: ['uglify']
      }
    },
    aerobatic: {
      // These are the files that should be deployed to the cloud.
      deploy: {
        src: ['index.html', 'dist/*.*', 'images/*', 'favicon.*'],
      },
			// The livereload works in conjuction with the livereload
			// server built into the watch task
      sim: {
        index: 'index.html',
        port: 3000,
        livereload: true
      }
    }
  });


  grunt.registerTask('build', ['jshint', 'uglify', 'cssmin']);

  // Specify the sync option to avoid blocking the watch task
  grunt.registerTask('sim', ['aerobatic:sim:sync', 'watch']);

  // Create a deploy alias task which builds then deploys to aerobatic in the cloud
  grunt.registerTask('deploy', ['build', 'aerobatic:deploy']);

  grunt.loadNpmTasks('grunt-aerobatic');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-watch');
	// Load other tasks
};
```  

## Simulator Mode
The simulator mode is a unique feature of Aerobatic that provides a fully integrated development environment. Typically when building static apps you work off of a localhost URL. This works well for simple apps, but there are scenarios where it breaks down. For example when registering with an OAuth provider you have to specify the callback URL that the provider will redirect to and this URL has to match the origin of the request from your actual app, not `localhost`. Another potential scenario is making a CORS request to a remote domain that has rules on what domains are allowed. By developing and testing on your production domain, you eliminate these and other potential environment discrepancies and uncover integration issues early on.

You start the simulator with the simple command `grunt aerobatic:sim --open`. This uploads your `index.html` file to the Aerobatic cloud platform where it is stored in a private cache that is specific to your individual Aerobatic userId. This way you are still fully isolated from live traffic or any other developers. The sim task watches the local `index.html` file and will automatically upload a new copy whenever it changes.

 The port number can be overridden in the grunt config if you choose. The URL that you point to during development is the same as your production app URL but with some additional query parameters:
 ```
http:<your_app_name>.aerobaticapp.com?sim=1&user=<your_user_is>&port=3000&reload=1
```
Rather than have to type this all in, you can simply pass the `--open` command line option which will launch a browser to the right fully formed URL.

When Aerobatic sees the `sim=1` in the URL it knows that the request should be treated differently than a normal request. The cached index.html for your userId is grabbed from cache and undergoes the same basic processing as a regular request by removing any `data-build-type="release"` elements and fixing any referenced assets to http://localhost:3000 (the port number can be overridden in the Gruntfile). Additionally a banner overlay is injected into `index.html` that appears in the top right corner of the page which lets you easily see at a glance that you are in simulator mode.

### Livereload
To power a true first-rate development workflow, it is possible to combine the simulator with [grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch) and [livereload](https://github.com/gruntjs/grunt-contrib-watch#optionslivereload) so your browser automatically refreshes whenever code changes are saved locally. Just define an alias task called `sim` that runs both `aerobatic:sim:sync` and `watch`. The `sync` option tells the `sim` task to run synchronously so as not to block the `watch` task from executing.

```js
grunt.registerTask('sim', ['aerobatic:sim:sync', 'watch']);
```
Now you can fire up the simulator by simply typing: `grunt sim --open`. Aerobatic will automatically inject the livereload integration script into `index.html`:

```html
<script src="//localhost:35729/livereload.js"></script>
```

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

## Deploying
Deploying code to production couldn't be any easier.

```bash
> grunt deploy --name '1.2.1' --message "Implemented user story"
```
The above assumes you have setup a grunt alias task which first builds your assets then invokes `aerobatic:build`.

```js
grunt.registerTask('deploy', ['build', 'aerobatic:deploy']);
```

The `--name` and `--message` arguments are optional. If you do not specify a `--name` a timestamp based value will be auto-generated for you.

Only the files matching the patterns in the grunt config are deployed to the cloud. Typically these will be your built assets, i.e. compressed and combined JavaScripts and stylesheets. However you could also choose to deploy your original source files. Always be sure to include your `index.html` in the `src` array.

```js
deploy: {
  src: ['index.html', 'dist/*.*', 'images/*', 'favicon.*']
}
```

By default this will only stage the new version, but not cause any live traffic to be directed to it. In your app dashboard you can adjust the [Traffic Control](/docs/traffic-control) rules to direct all or a portion of live traffic to this version. Just because a new version has been deployed doesn't mean that it will ever go live. One of the powerful aspects of the Aerobatic deployment model is that every version ever deployed is accessible via a special URL like `http://<my_app>.aerobaticapp.com?version=<version_id>`. You could use these preview URLs for testing versions in a completely integrated manner and only direct live traffic if the version gets the green light from stakeholders.

Because the deploy process is entirely command line driven you could easily integrate it into an automated continuous delivery process. Perhaps you have a version numbering convention and your build process could inject the dynamic version number as the `--name` argument.

All this version management and workflow is great, but sometimes you just want to bypass all that ceremony and get your changes live. For that reason the `deploy` task has a `--cowboy` switch that will force all traffic to automatically be directed to the new version.

```bash
> grunt deploy --cowboy
```

Once the deploy completes, the grunt task will report back the URL where the new version can be accessed. You can even specify `--open` to automatically launch the version in a new browser tab.

```bash
> grunt deploy --open
```

If you specify both the `--cowboy` and `--open` options you'll need to explicitly assign the value `true` due to a limitation in how grunt interprets boolean command line options. Rather than the preview URL, this will launch the primary app URL since the new version is now live.

```bash
> grunt deploy --open=true --cowboy=true
```
