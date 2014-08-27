<meta id="blogShortUrl" value="http://bit.ly/1rb6RTo">
<meta id="blogAuthorBio" value="David Von Lehman is co-founder of Aerobatic. He oftentimes has visions of JavaScript, cloud platforms, and single page applications dancing in his head. Reach him at @davidvlsea or david@aerobatic.io">

Starting a brand new single page application from scratch can be daunting. In order to make it as easy as possible to go from zero to something interesting on the Aerobatic platform, we're releasing starter templates as GitHub repos for the three leading JavaScript application frameworks: [AngularJS](https://www.angularjs.org), [Ember](http://emberjs.com), and [Backbone](http://backbone.js). When you create a new Aerobatic app, you can choose your preferred framework and get instructions on how to clone the appropriate repo to use as a starting point.

<img class="img-responsive" src="https://s3-us-west-2.amazonaws.com/aerobatic-media/aerobatic-starter-template-screenshot.png">

Each repo attempts to bake in commonly accepted best practices, optimizations, naming conventions, etc. for each respective framework. Oftentimes sample projects found online purposefully ignore best practices like representing each module as its own file, minifying and concatenating files for production builds, etc. In the context of a tutorial this can help convey the basic code concepts, but for a real world app you want to take measures to ensure the codebase will be maintainable and configured for optimal performance. Each of our starter repos come with a preconfigured Gruntfile that:
* Combines all scripts into a single minified download
* Combines all stylesheets into a single minfied download
* Precompiles html templates into JavaScript
* Sets up a watch so that build actions are taken and the browser reloads whenever source files change
* Configures all unit tests to execute via `grunt test`
* Declares the [grunt-aerobatic](https://www.npmjs.org/package/grunt-aerobatic) task with shortcuts for running the development simulator and deploying to the cloud

Additionally each repo has a bower.json file for managing client dependencies.

The READMEs of each respective repo has more details on how the application is structured and what Grunt tasks are used to power the development workflow.

Name  | Repo | Live Demo
------------- | -------------
Angular Seed  | https://github.com/aerobatic/angular-seed/ | http://angular-seed.aerobaticapp.com/
Backbone Boilerplate | https://github.com/aerobatic/backbone-boilerplate | http://backbone-boilerplate.aerobaticapp.com/
Ember Starter | https://github.com/aerobatic/emberjs-starter-kit | http://emberjs-starter-kit.aerobaticapp.com/
Vanilla JS | https://github.com/aerobatic/starter-template | http://starter-template.aerobaticapp.com/


The possibilities for configuring how a single page app is structured and how the build process works are endless, but we've attempted to incorporate the opinions and guidelines of leading members of the open-source JavaScript community. Feel free to fork any of the repos and customize them for your own purposes or suggest changes and issue a pull request.

One thought for the future is to encapsulate most of the grunt configuration in an npm module. That way you'd be able to dramatically simplify your own project's Gruntfile by delegating the boilerplate to a function call so long as your project conforms to certain conventions - convention over configuration. Let me know if this is something that you'd find valuable.

Hopefully these starter templates will smooth the process of getting a new app up and running on the Aerobatic platform. The goal is for you to spend less time on configuration setup and get right to building app experiences.

Happy Coding!
