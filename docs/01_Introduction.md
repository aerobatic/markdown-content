
#Introduction

Welcome to Aerobatic, the cloud platform for front-end developers that makes it fun to build nimble HTML5 web apps in record time. So what is Aerobatic? In a nutshell, it's a platform as a service (PaaS) for HTML5 web apps. You could think of it as [Heroku](http://www.heroku.com) for the front-end. Like Heroku, the Aerobatic platform makes it super simple to deploy and host an app in the cloud. Like this easy:

```bash
> grunt deploy --name '2.1.1'
```

Unlike Heroku, Aerobatic does not require developers to build and maintain a backend with Ruby, Node, Python, etc. Developers can build powerful apps using only browser-based technologies like Javascript, HTML, and CSS.

Hold on, you might be thinking, this sounds an awful lot like static pages that can simply be hosted on [Amazon S3](http://aws.amazon.com/s3) or [GitHub Pages](http://pages.github.com). Aerobatic provides the same ease of deployment and cloud level scale as those platforms, but layers on a suite of smart hosting services that provide a much greater level of functionality, performance, and security than can easily be achieved with with pure static pages. We think that these intelligent hosting modules, along with our [Grunt](http://gruntjs.com) based workflow, provide a powerful foundation that allows developers to focus more energy on crafting apps and less on infrastructure, configuration, and environment setup.

## Nimble App Delivery
We all know how critical agility is when it comes to app delivery. It is expected that teams be able to build quickly, react to changing requirements, perform UX experimentation, and fix issues with minimal turnaround. The Aerobatic platform enables nimble app delivery by cutting out as much waste from the process as possible while still providing measures to mitigate the risks of moving fast.

Environment management is a notorious source of waste; maintaining one or more lower environments that accurately mirror production is a big time sink. Aerobatic uses a feature known as [Simulator Mode](#!/docs/developing-apps) to enable developers to code locally, but see their changes in the browser fully integrated with the production server platform in the cloud. Working in this style minimizes configuration and developer setup, and surfaces most integration issues immediately, rather than once in production. Deploying to the cloud is a simple command line call to "grunt deploy --name 'version name'". Developers can deploy themselves, or the command can be integrated into a continuous delivery pipeline with tools like [Jenkins](http://jenkins-ci.org/), [Go](http://www.thoughtworks.com/products/go-continuous-delivery),
[Travis CI](https://travis-ci.org/), and others.

By default, when new versions are deployed to the cloud, they won't automatically receive live traffic. Aerobatic includes a feature called [Traffic Control](#!/docs/traffic-control) which allows split traffic management that proportionally directs new sessions to one of multiple app versions - all configured in the dashboard using simple slider controls. This is a powerful capability, sometimes known as canary or [blue/green](http://martinfowler.com/bliki/BlueGreenDeployment.html) deployments, that to date has primarily been confined to top technology companies such as Facebook, Netflix, and Etsy due to the high degree of infrastructure automation required to pull it off with server-side app delivery. But due to the client-side app delivery model, Aerobatic is able to provide this right out of the box. Traffic control can be used both to A/B test or as a way to mitigate risk by initially exposing only a small subset of traffic to new versions.

##Single Page Applications
Modern web apps no longer need be constrained by the traditional one page at a time with a full refresh paradigm. Mobile devices have created the expectation of a richer interaction style with smooth state transitions. While it has been possible to build browser based apps where all interaction takes place within the context of a single page load for years, only relatively recently have the right combination of factors converged in order to make building HTML5 single page applications accessible to a wide audience of web developers. These factors include high market penetration of modern browser versions with fast JavaScript execution, CSS3 for highly performant animations, and robust client app frameworks such as  [Angular](https://angularjs.org/), [Ember](http://emberjs.com/), [Backbone](http://backbonejs.org/), and others.

The Aerobatic platform is designed specifically for building and hosting single page applications using any JavaScript or CSS library. It provides a streamlined developer workflow and a suite of cloud-based smart hosting modules that are highly complementary to your custom app code running in the browser. As single page web apps emerge as the default style of web app development, Aerobatic strives to be the premier platform on which they are delivered.

##Smart Hosting Modules
You can go a long way with just the browser and static pages these days. With AJAX, [CORS](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS), web APIs, backend as a service offerings such as [Firebase](https://www.firebase.com/) and [Parse](https://parse.com/docs/js_guide), it's increasingly possible to build sophisticated web apps with no server component at all. However despite these advances, the pure static app model does introduce complications and forces certain compromises to be made.

For example a fully static app that invokes a web API requiring access keys (as most do), those keys need to be sent in the HTML payload to the browser in plain site. Aerobatic provides an [Intelligent API Proxy](#!/backend-integration) that will make the API call on the app's behalf, tacking on the access keys which are securely stored on the server. Additionally if the API being consumed has high latency or is rate limited, the proxy can be set to cache the responses in the cloud for more predictable performance.

Another capability that can be difficult to achieve in the browser alone is authentication. Aerobatic makes it super simple to require users to authenticate via multiple popular OAuth providers including [Google](https://developers.google.com/accounts/docs/OAuth2), [LinkedIn](https://developer.linkedin.com/documents/authentication), [Facebook](https://developers.facebook.com/docs/facebook-login/login-flow-for-web/), [Twitter](https://dev.twitter.com/docs/auth/using-oauth), [Instagram](http://instagram.com/developer/), [GitHub](https://developer.github.com/v3/oauth/). All developers need to do is register their application with the desired provider and securely store their assigned client key and secret keys within the app dashboard. Aerobatic will automatically take care of the entire OAuth handshake exchange.

<!---
Another challenge faced by single page applications has to do with SEO. Because content is dynamically loaded via JavaScript calls rather than as part of the initial page load, search engine crawlers won't see all the content. Aerobatic provides an [SEO Enhancement Module](#!/docs/seo) which allows even single page applications to be fully discoverable by search engines.
--->

##Performance
The key to achieving great performance for web apps is efficient delivery of the client assets to the browser. Aerobatic automatically utilizes all of the industry best practices for optimizing client assets delivery. Your Gruntfile will configure the steps to prep your assets for deployment including pre-processing and compression. When you run `grunt deploy` your prepared assets are written to S3 and the references to these resources are modified in your HTML to point to a CDN. Additionally a unique string representing the version of the deployed app is included in the absolute resource URLs - a process known as [fingerprinting](https://developers.google.com/speed/docs/best-practices/caching#LeverageBrowserCaching).

```html
<script src="//d2q4nobwyhnvov.cloudfront.net/<app_id>/<version_key>/dist/app.min.js"></script>
```

This allows very aggressive cache headers to be set so assets are cached at all levels from the CDN all the way to the browser. Having the URLs for the client assets be unique for each version is what allows multiple versions of the application to be live at once.

Oftentimes in app delivery, optimization efforts that have a major impact on the quality of the user experience get deferred because developers are 100% focused on adding functionality. With Aerobatic,  optimization is included from day 1, avoiding the need for a special effort to revisit performance best-practices down the road.
