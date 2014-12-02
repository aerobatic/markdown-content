# Introduction

Welcome to Aerobatic, the cloud platform for front-end developers that makes it fun to build nimble HTML5 web apps in record time. So what is Aerobatic? In a nutshell, it's a platform as a service (PaaS) for single page web apps. You could think of it as [Heroku](http://www.heroku.com) for front-end client apps.

However, unlike Heroku, Aerobatic does not require developers to build and maintain a backend with Ruby, Node, Python, etc. Powerful applications can be built using only browser-based technologies, namely Javascript, HTML, and CSS. 
In this respect Aerobatic provides similar functionality to static hosting with [Amazon S3](http://aws.amazon.com/s3) or [GitHub Pages](http://pages.github.com). Aerobatic provides the same ease of deployment and cloud level scale as those platforms, but layers on a management dashboard and a suite of smart hosting services that provide a greater level of functionality, performance, and security than can easily be achieved with with pure static pages. 

We think that these intelligent hosting modules, along with our [yoke](/docs/yoke) command line tool, provides a simple yet powerful workflow allowing front-end developers to focus more energy on crafting apps and less on infrastructure, configuration, and environment setup.

##Single Page Applications
Modern web apps no longer need be constrained by the traditional one page at a time with a full refresh paradigm. Mobile devices have created the expectation of a richer interaction style with smooth state transitions. While it has been possible to build browser based apps where all interaction takes place within the context of a single page load for years, only relatively recently has the right combination of factors converged that make building HTML5 single page applications accessible to a wide audience of web developers. These factors include high market penetration of modern browser versions with fast JavaScript execution, CSS3 for highly performant animations, and robust client app frameworks such as  [Angular](https://angularjs.org/), [Ember](http://emberjs.com/), [Backbone](http://backbonejs.org/), along with promising emerging ones such as [React](http://facebook.github.io/react/) and [Polymer](https://www.polymer-project.org/).

As single page web apps emerge as the default style of web app development, Aerobatic strives to be the premier platform on which they are delivered.

## Smart Hosting Modules
While pure client code can accomplish an awful lot, there are still scenarios where it pays to have a server working on behalf of your client app. The Aerobatic smart hosting modules do just that without you having to write any server code at all.

### API Gateway
While it is possible to invoke remote APIs directly via [CORS](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS), there are many situations where the target API cannot be directly called from client script, for security reasons, or lack of CORS support. 

Many APIs require one or more keys or tokens to be passed along in the URL or in a HTTP header. Depending on the API, you may not wish to leak these keys to the world by sending them in clear text in your JavaScript. Aerobatic provides an [API Gateway](/docs/api-integration) that will proxy the API call on the app's behalf while injecting the access keys that are securely stored on the server. Additionally if the API being consumed has high latency or is rate limited, the proxy can be set to cache the responses in the cloud for more predictable performance.

### Authentication
Another capability that can be difficult to achieve in the browser alone is authentication. Aerobatic makes it super simple to require users to authenticate via multiple popular OAuth providers including [Google](https://developers.google.com/accounts/docs/OAuth2), [LinkedIn](https://developer.linkedin.com/documents/authentication), [Facebook](https://developers.facebook.com/docs/facebook-login/login-flow-for-web/), [Twitter](https://dev.twitter.com/docs/auth/using-oauth), [Instagram](http://instagram.com/developer/), [GitHub](https://developer.github.com/v3/oauth/). All developers need to do is register their application with the desired provider and securely store their assigned client key and secret keys within the app dashboard. Aerobatic will automatically take care of the entire OAuth handshake exchange.

## SEO
Another challenge faced by single page applications has to do with SEO. Because content is dynamically loaded via JavaScript calls rather than as part of the initial page load, search engine crawlers won't see all the content. Aerobatic provides an [Snapshot module](/docs/seo) which allows even single page applications to be fully discoverable by search engines.

It takes less than a minute to create and deploy your first app, go ahead and [give it a shot](/docs/getting-started).

