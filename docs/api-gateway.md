# API Gateway

In the single page app model the browser generally integrates with backend APIs by invoking remote URLs directly. Traditionally the browser security model intentionally disallows many forms of network communicate across domain boundries. There are workarounds like JSONP and iframe hacks, but they're not first class browser primitives for cross domain communication. Fortunately HTML5 introduces [cross-origin resource sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (aka CORS) which enables the same XmlHttpRequest that is used for same domain AJAX calls to work across domains. While CORS is a key technology for enabling browsers as application platforms, there are still drawbacks to contend with.

Perhaps the most glaring of these drawbacks is that any credentials needed to communicate with the remote API must be included in the web page where it can be seen by anyone with access to the site. For some APIs this might not be a big deal, but where access to the API is restricted, it could be unacceptable.

Another drawback to direct communication from the browser to a remote API is performance. If the API has poor response times every end user will have to incur that latency. The API might be rate-limited and only allow a certain number of calls per period of time.

## Intelligent Service Proxy
To combat these challenges, Aerobatic offers the service proxy module. Rather than making network calls directly to the remote service, your JavaScript makes a simple AJAX call back to an endpoint /proxy on the same host as the app. The actual URL of the API is encoded in a query parameter. Any sensitive values such as access keys, passwords, etc. are tokenized in the form `@@CLIENT_KEY@@`. These tokens can appear anywhere in the request including the querystring, request body, or HTTP headers. The proxy will replace the tokens with corresponding config values that are setup in the Aerobatic app dashboard.

Here's how the service proxy could be invoked with jQuery. In this example the proxy would make the call to `https://api.someservice.com/endpoint?access_key=my_access_key`. The value "my_access_key" is stored in the SOMESERVICE_API_KEY config key and never sent over the wire to the browser.

```javascript
$.ajax({
  url: "/proxy",
  type: "GET",
  data: {
    url: "https://api.someservice.com/endpoint?access_key=@@SOMESERVICE_API_KEY@@"
  },
  success: function(data, status) {
  }
});
```

### Authorization Header
If you need to pass an Authorization header to an API, you need to pass it to the proxy with the name `X-Authorization`. The Aerobatic proxy will automatically do any variable substitution then forward the header on as just plain `Authorization`.

```js
$.ajax({
  type: "GET",
  url: "/proxy?url=" + encodeURIComponent("https://api.someservice.com"),
  dataType: 'json',
  beforeSend: function(xhr) {
    xhr.setRequestHeader ("X-Authorization", "Basic @@SOMESERVICE_AUTH_HEADER@@");
  },
  success: function (response){
  }
});
```

For APIs that use HTTP Basic Authentication, it is common for the username and password to be concatenated together then base64 encoded. In order to store this as a single config setting you need to perform the base64 encoding first and store the final value in the config. An easy way to do this is in your browser's developer console with the following command:

```js
btoa("<username>:<password>");
```

### Caching
Another feature of the service proxy is caching support. By appending the parameter _cache=1_ to the /proxy query string you can tell Aerobatic to cache the API response. If the response is already in the cache, the remote API never gets invoked. In addition to the _cache_ parameter, a key parameter must also be specified which is the string that should uniquely identify the API response in the cache. A third _ttl_ parameter can be specified which dictates how many seconds the response should be held in cache. If no _ttl_ parameter is specified, the proxy will attempt to honor either the _max-age_ or _expires_ headers returned by the API.

Here's the same example from above with caching enabled:

```javascript
$.ajax({
  url: "/proxy",
  type: "GET",
  data: {
    url: "https://api.someservice.com/endpoint?access_key=@@SOMESERVICE_API_KEY@@",
    cache: 1,
    ttl: 300 // cache for 5 minutes
  },
  success: function(data, status) {
  }
});
```

If the proxy locates an API response in the cache, it will automatically set the _max-age_ header in the response back to the browser to the time remaining. So for example let's assume the app cache is empty. User A makes a call to the proxy for url X at 11:00 and a ttl of minutes. The proxy would make the call to the remote API, pipe the response first to the cache then on to the client with a _max-age_ header of 300 seconds. This will force user A's browser to cache the response so any additional calls user A makes for url X in the next 5 minutes will not incur any network round trip at all. Now say at 11:02 user B's browser makes a proxy call for url X. The proxy pipes the previously cached response to the browser setting the _max-age_ header to 3 minutes since that is how much longer it has to live.  

In this way the Aerobatic service proxy can provide the benefits of caching even if the remote API does not set appropriate caching headers.

__Note__: the jQuery $.ajax function will automatically encode strings in the data object. If you are constructing the API URL from scratch you should use the [encodeURIComponent](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) function to encode it.
