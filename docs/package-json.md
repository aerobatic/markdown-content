# Configuring package.json

Rather than introduce a new special configuration file, Aerobatic makes use of the [package.json](https://www.npmjs.org/doc/files/package.json.html) file used by [npm](https://www.npmjs.org). If you're new to node.js, npm is the package manager for node and is included as part of the node.js installation. Chances are you will be using a build tool such as grunt or gulp as part of your development workflow, both of which require node.js and npm. 

The yoke command line tool reads several of the standard npm attributes and also expects a custom `_aerobatic` section to be defined that, at a minimum, specifies the Aerobatic appId.

Let's take a look at a typical package.json:

```json
{
  "name": "your app name", 
  "version": "1.0.0",
  "scripts": {
    "build": "grunt build",
    "watch": "grunt watch"
  },
  "dependencies": {
  },
  "devDependencies": {
  },
  "_aerobatic": {
    "appId": "<your_aerobatic_app_id>"
  }
}
```

### name
The name of your Aerobatic app. This is set automatically by yoke when running the `app:create` command.

### version
When deploying code, yoke defaults the version name to this attribute. 

### scripts.build 
When `yoke deploy` is run you are prompted whether to run the build step. If yes, then the script specified here is executed. You can call a build tool with a value such as `grunt build` or `gulp build`, or you could run any arbitrary command line script.

### scripts.watch
When `yoke serve` or `yoke sim` is run, it will automatically run this script. Typically this script will just be a passthrough to your build tool such as `grunt watch` or `gulp watch`. Note that Aerobatic is able to automatically pre-process certain syntaxes 

### _aerobatic
Custom Aerobatic section. The underscore prefix is used in accordance with best practices for extending package.json to avoid any possible future collisions.

### _aerobatic.appId
Required attribute that identifies the unique Aerobtic appId. If this value is missing, it can be restored by running `yoke app:bind`.
