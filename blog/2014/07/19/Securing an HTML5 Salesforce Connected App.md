<meta id="blogAuthor" value="david" />
<meta id="blogShortUrl" value="http://bit.ly/WAK9L2" />

Recently we added a Salesforce connected app to the [Aerobatic app gallery](#!/gallery).
Not surprisingly security is a major concern when it comes to Salesforce
data as it contains valuable company and personal information. While developing our
sample app, I discovered there are a number of decisions to be made as to how
security is implemented. This post shares what I learned and points out some of
the different options available.

_Note this post is specific to standalone connected Salesforce apps rather than Apex apps which runs atop the Force.com platform._

Our sample app, [Salesforce Contact Manager](https://sfcontacts.aerobaticapp.com/),
simply displays all your Salesforce contacts as business cards. As always the full
source code is available [on GitHub](https://github.com/aerobatic/sfcontacts).
The app is implemented with AngularJS, but the information presented is
applicable to any JavaScript powered HTML5 app. If you don't already have an
account, the [Developer Edition](https://developer.salesforce.com/signup)
is free and comes seeded with several sample contacts.

## Creating a Connected App
The first step is to login to Salesforce and [create a new Connected App](https://help.salesforce.com/apex/HTViewHelpDoc?id=connected_app_create.htm&language=en_US).
In this post we're concentrating on the OAuth setup, so ensure the __Enable OAuth Settings__
box is checked. In the __Callback URL__ you'll need to enter the full URL of your app
which the documentation states: __must use secure HTTP (HTTPS)__.
This means that if you are running your app on localhost during development, you must create a
self-signed certificate. Searching Google for how to create a self-signed certificate
that Chrome will always trust led me to a number of dead ends. Finally [this article](http://www.robpeck.com/2010/10/google-chrome-mac-os-x-and-self-signed-ssl-certificates/#.U8mfZI1dUgo)
did the trick on OSX. This is only a concern while in development, once you
deploy to production you'll want to use a certificate provisioned by a trusted authority.

One downside to developing your app on `https://localhost` is that you'll likely need to
create two connected apps: one for development using localhost as the callback and a second with
your production URL as the callback. One of the benefits of building HTML5 apps
on the Aerobatic platform is the [simulator mode](#!/docs/getting-started?simulator-mode)
that enables running a development environment directly from the production URL. This
way you code in a fully integrated mode and only have to setup a single Salesforce app. Your Aerobatic OAuth callback URL is in the form: `https://your_app.aerobaticapp.com/auth/callback`.

After clicking Save, be sure to make note of your Consumer Key and Consumer Secret.

## Implementing OAuth
Now that we have an app, we need authentication wired up as that is a prerequisite for making API calls. Salesforce supports three different
flavors, described in [this article](https://www.salesforce.com/us/developer/docs/api_rest/Content/intro_understanding_authentication.htm) as follows:

1. [Web server flow](https://www.salesforce.com/us/developer/docs/api_rest/Content/intro_understanding_web_server_oauth_flow.htm), where the server can securely protect the consumer secret.
2. [User-agent flow](https://www.salesforce.com/us/developer/docs/api_rest/Content/intro_understanding_user_agent_oauth_flow.htm), used by applications that cannot securely store the consumer secret.
3. [Username-password flow](https://www.salesforce.com/us/developer/docs/api_rest/Content/intro_understanding_username_password_oauth_flow.htm), where the application has direct access to user credentials.

The third option requires the user to enter their credentials in your own app;
defeating the point of OAuth which is to get out of the password business entirely. The other two options are viable, both with their own pros and cons as we'll see.

### User-Agent Flow
This flow is implemented entirely in the browser, the web server does not get
involved at all. The user's browser loads a page (either full window or popup) pointing to the URL:

```
https://login.salesforce.com/services/oauth2/authorize?redirectURI=https://yourappurl/oauthcallback.html
```
After the user logs in via the Salesforce controlled page the window is
redirected to the URL specified by the `redirectURI` query parameter. This URL must be __HTTPS__ and needs to match the domain of the __Callback URL__ specified when the app was setup in Salesforce.

On the `oauthcallback.html` page, JavaScript is used to extract the `access_token` out of the query string and store
it somewhere to be used in future API calls. The [Salesforce documentation](https://www.salesforce.com/us/developer/docs/api_rest/Content/intro_understanding_user_agent_oauth_flow.htm)
states that: __this token should be protected as though it were user credentials__.
In other words, don't store it in a cookie. Stashing it in a JavaScript variable
in memory is more secure, but it means that a page refresh will necessarily require re-authenticating since the token will be flushed from memory.

There's a good [sample app](https://github.com/ccoenraets/soql-explorer/blob/master/client/js/app.js) on GitHub
from [@ccoenraets](https://github.com/ccoenraets) that
demonstrates this approach:

```js
var apiVersion = 'v30.0',
    clientId = 'YOUR_CONSUMER_KEY_HERE',
    loginUrl = 'https://login.salesforce.com/',
    redirectURI = "https://localhost:3000/oauthcallback.html",
    proxyURL = 'https://localhost:3000/proxy/',
    client = new forcetk.Client(clientId, loginUrl, proxyURL);

function login() {
    var url = loginUrl + 'services/oauth2/authorize?display=popup&response_type=token' +
        '&client_id=' + encodeURIComponent(clientId) +
        '&redirect_uri=' + encodeURIComponent(redirectURI);
    popupCenter(url, 'login', 700, 600);
}

function oauthCallback(response) {
    if (response && response.access_token) {
        client.setSessionToken(response.access_token, apiVersion, response.instance_url);
    } else {
        alert("AuthenticationError: No Token");
    }
}
```

The `forcetk` object is defined in the [Salesforce JavaScript REST Toolkit](https://github.com/developerforce/Force.com-JavaScript-REST-Toolkit),
a javascript library included in the page. The `setSessionToken` function
simply stashes the `access_token` in memory and includes it in an authorization
header in subsequent API calls.

Here's the source of the `oauthcallback.html` page that Salesforce passes
the `access_token` back to in the URL:
```html
<html>
<body>
<script>
if (window.location.hash) {
    var message = decodeURIComponent(window.location.hash.substr(1)),
        params = message.split('&'),
        response = {};
    params.forEach(function (param) {
        var splitter = param.split('=');
        response[splitter[0]] = splitter[1];
    });
    window.opener.oauthCallback(response);
    window.close();
}
</script>
</body>
</html>
```

One thing to notice is that the app's `CONSUMER_KEY` has to be included in
the client code in plain site. As we'll see shortly, the Aerobatic approach
to OAuth allows storing the consumer key securely on the server, avoiding exposing it over the network.

### Web Server Flow
The [web server flow](https://www.salesforce.com/us/developer/docs/api_rest/Content/intro_understanding_web_server_oauth_flow.htm)
works much the same way as the user-agent flow, but the OAuth negotiation takes place between Salesforce and the host web server rather than the user's browser.
The advantage to this approach is the `access_token` can be securely
stored on the server. Note the `access_token` would still need to be passed to the client if API calls were to be made directly from the browser, but as I'll cover shortly, Salesforce doesn't allow this type of API invokation.

The downside to this approach is that you'll need to write additional code on
the server to handle this. If you have a Node.js/Express app, there is a great
package called [Passport](http://passportjs.org/) from
[@jaredhanson](https://github.com/jaredhanson) that provides a plugin framework
for integrating with a [wide variety](http://passportjs.org/guide/providers/)
of OAuth providers. Salesforce developers have contributed the
[passport-forcedotcom](https://github.com/joshbirk/passport-forcedotcom) npm
module that plugs into Passport to enable the web server flow for Salesforce apps. The Aerobatic platform, which is implemented in Node and Express, makes use of this module.  

## OAuth with Aerobatic
With Aerobatic you get the security benefits of the web server flow without
having to author any authentication specific server or client code at all.
In the app dashboard, just select the Salesforce OAuth provider and paste
in your consumer key and secret.

<img class="img-responsive" src="https://s3-us-west-2.amazonaws.com/aerobatic-media/aerobatic-oauth	-setup-screenshot.png" alt="Aerobatic OAuth Screenshot" />

For OAuth protected Aerobatic apps, we deviate from the single page app model
slightly by introducing a second page `login.html`, which serves as a
pre-authenticated landing page. Both pages are served from the same root URL: `https://your_app.aerobaticapp.com/`.
Behind the scenes Aerobatic serves the correct html page based on the user's authenticated state. The `login.html` page should include a relative link to the URL `/auth` inviting the user to sign-in with Salesforce.

```html
<a href="/auth" class="btn btn-primary btn-lg btn-block">Sign In with Salesforce</a>
```
The Aerobatic platform will automatically initiate the OAuth handshake with the
configured provider and store the `access_token` in session state in the
cloud. A secure session cookie is issued to the browser to identify the logged-in
user on subsequent calls to the API proxy (covered next). This secure
cookie based approach is the tried and tested mechanism that
nearly every login protected website on the internet relies upon including
eCommerce, online banking, and so on.

Within your app's JavaScript, the logged in Salesforce user is accessible via
the `__config__.user` variable. The format of the object conforms to the
[PassportJS normalized user profile](http://passportjs.org/guide/profile/) schema.

## Making API Calls
Now that the user is authenticated via Salesforce and the app has an access
token, we can finally get to the interesting work of making API calls. Somewhat
surprisingly given the level of sophistication of the developer platform, the
Salesforce APIs are not CORS enabled, so API requests initiated via Ajax in the
browser must be routed through a proxy. This proxy could live on the origin host of the app or it could be a CORS specific proxy on a separate domain. [James Ward](http://www.jamesward.com/) of Salesforce has written an [open-source sample CORS proxy](http://www.jamesward.com/2014/06/23/cross-origin-resource-sharing-cors-for-salesforce-com)
built specifically for the Salesforce REST API.

You can also implement this yourself if you have control over the server
code where your app is hosted. Node.js is very well suited for this use case;
so much so you can implement it in [one line](https://github.com/ccoenraets/soql-explorer/blob/master/server.js).
The Salesforce REST Toolkit includes a [php proxy](https://github.com/developerforce/Force.com-JavaScript-REST-Toolkit/blob/master/proxy.php) also.

### Aerobatic API Proxy
Aerobatic has a [built-in proxy](#!/docs/backend-integration) that offers some additional capabilities.
As discussed earlier, with the Aerobatic OAuth integration,
the access token is not available to the browser. In the `X-Authorization`
header we use a token in the form `@@user.accessToken@@`. The proxy
will substitute any token pattern by evaluating the expression nested between
the `@@` symbols.

```js
var soql = "SELECT Name, Title, Phone, Email FROM Contact";
var url = "https://na17.salesforce.com/services/data/v30.0/query?q=" + encodeURIComponent(soql);

var config = {
  method: 'GET',
  url: '/proxy?url=' + encodeURIComponent(url),
  headers: {
    'X-Authorization': 'OAuth @@user.accessToken@@'
  }
};

$http(config).success(function(data, status, headers, config) {
  contacts = data.records;
}).
error(function(data, status, headers, config) {
  $log.error(data);
});
```
The proxy also offers caching capabilities that can have a dramatic performance
improvement in the scenario where an app makes the same API call with the same
parameters for all users. For example in my [previous blog post](#!/blog/2014/07/08/build-a-publishing-platform-with-angularjs-github-and-markdown)
I showed how the publishing system on www.aerobatic.io works. The blog post you are reading right now was retrieved from GitHub and cached within the Aerobatic proxy.

## Wrapping Up
I've covered the two basic aspects of security in a Salesforce connected app:
authenticating via OAuth, and making API calls passing along the OAuth access token. Hopefully you've picked up something useful and perhaps you'll consider building your next HTML5 Salesforce connected app with Aerobatic.

Happy coding!
