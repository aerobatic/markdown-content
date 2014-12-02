# Performance

The key to achieving great performance for web apps is efficient delivery of the client assets to the browser. Aerobatic automatically utilizes most all industry best practices for optimizing client asset delivery. Your build step is responsible for preparing assets through minification, concactenation, etc. but Aerobatic takes care of the rest. 

When you run `yoke deploy` your prepared assets are written to S3 and the references to these resources are modified in your HTML to point to a CDN. Additionally a unique string representing the version of the deployed app is included in the absolute resource URLs - a process known as [fingerprinting](https://developers.google.com/speed/docs/best-practices/caching#LeverageBrowserCaching).

```html
<script src="//d2q4nobwyhnvov.cloudfront.net/<app_id>/<version_key>/dist/app.min.js"></script>
```

This allows very aggressive cache headers to be set so assets are cached at all levels - from the CDN all the way to the browser. Having the asset URLs be unique for each version is what allows multiple versions of the application to be live at once (see [traffic control docs](/docs/traffic-control)).

The index page is not served from a CDN, but it does utilize etags, so a request that would result in the same response content will instead return a lightweight `304 Not Modified` status code.

Oftentimes in app delivery, optimization efforts that have a major impact on the quality of the user experience get deferred because developers are 100% focused on adding functionality. With Aerobatic,  optimization is included from day 1, avoiding the need for a special effort to revisit performance best-practices down the road.
