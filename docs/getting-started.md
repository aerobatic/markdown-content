# Getting Started
Let's get your app up and running on Aerobatic. It should only take a few minutes.

## Install yoke
The `yoke` command line tool is used to power the development workflow including creating, building, and deploying application. Open your terminal and run the following:

```
npm install -g yoke-cli
```

## Logging in
If this is the first time using yoke, you'll need to login by running the following:

```
yoke login
```

Your userId and secretKey can be found on the profile page: [https://portal.aerobaticapp.com/profile](https://portal.aerobaticapp.com/profile)

## Creating an application
To create a new application, simply type:

```bash
yoke create-app
```

The terminal interface will walk you through a short set of questions such as the name and optionally a starter template that will provide a solid scaffolding or various JS frameworks including [Angular](https://github.com/aerobatic/angular-seed), [Ember](https://github.com/aerobatic/emberjs-starter-kit), and [Backbone](https://github.com/aerobatic/backbone-boilerplate). If a starter template is selected, the installer will download the latest code from GitHub and run `npm install` and `bower install` (if a `bower.json` file is found).

### Existing Codebase
You can also create an Aerobatic app from an existing codebase. Just select `Existing code` when prompted by the `create-app` command.


## Running app locally
Now that the app code is downloaded, you can launch the app in simulator mode using the following command:

```
yoke sim -o
```
The simulator mode is a hybrid development environment where the index page is served from your production URL, but your client assets are served from `http://localhost:3000` (or a port of your choosing). This gives you the best of both worlds: an isolated development sandbox, but also fully integrated with the Aerobatic platform and the internet at-large. The `-o` switch automatically opens a browser window to your app.


## Coding

The development server is automatically configured for [livereload](http://livereload.com/), so as you change files like .js and .html, the browser will automatically reload upon save. For .css files, changes will automatically get pulled into the page without incurring a full page reload. You do not need to configure livereload in your `Gruntfile.js` or `gulpfile.js`, or rely on a browser plugin, `yoke` will take of it. At this point development can proceed in the familiar pattern.


## Preview in release mode
Before you deploy your app, you'll want to run it in release mode locally to ensure that there are no errors in the built assets. You can preview the app in `release` mode locally by running the following:

```
yoke serve -o --release
```

By default `yoke` will look for release assets in a directory at the root of your repo called either `build` or `dist`. However if you have a different directory convention, you can explicitly specify which files should be deployed in the `_aerobatic` section of the `package.json` file.

## Deploying your app
After validating that everything looks good in release mode, you're ready to deploy, which is as easy as:

```
yoke deploy
```

You'll be prompted for a version name (which is pre-populated with the version attribute from `package.json`) and an optional message. All text based assets are gzip compressed and deployed to the Aerobatic cloud platform. In just a few seconds your app is live to the world!

## Next Steps
Now that you've got a basic app running, here's some further topics to explore to gain a deeper understanding of how Aerobatic works and add more functionality to your app:

* [Working with index.html](/docs/index-html)
* [Configuring package.json](/docs/package-json)
* [Simulator mode](/docs/simulator-mode)
* [App Authentication](/docs/auth)
* [API Gateway](/docs/api-gateway)
* [Search Engine Optimization](/docs/seo)
