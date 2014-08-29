Aerobatic has built-in support for requiring users to login via OAuth with a number of popular providers including Facebook, GitHub, Google, Twitter, Yammer, Instagram, and Salesforce. Once authenticated your app can make API calls to the provider on behalf of the logged user. The platform takes care of all the plumbing of integrating with the provider making it very simple to authenticate users of your app.

## OAuth Flows
In a web application there are two basic approaches that can be implemented: the web server flow and the browser flow. In the web server flow the server initiates the handshake passing along the client ID and secret to the provider then storing the access token that is passed back from the provider once the user has granted permission. The browser flow works much the same but as the name implies, JavaScript running in the browser is responsible for both initiating the auth flow and storing the token. A good high level overview can be found [here](http://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified).

Apps running on the Aerobatic platform can utilize either approach, however for the browser flow you have to write the code yourself (or use a client library). This article specifically covers the OAuth flow baked into the Aerobatic server platform which requires very little code to be authored.

##Implementation Steps
* In the settings for your app in the dashboard click OAuth and select your desired provider from the drop down.
* Register your application with the provider. The appendix below provides links to the pages where you can register your app.
* For the __callback URL__ enter the following: `http://<your_app_name>.aerobaticapp.com/auth/callback`.
* Copy the client key and secret and paste them into the corresponding fields in the Aerobatic dashboard.
* In your app repo, create a file `login.html` in addition to `index.html`. The `login.html` page represents the portion of your app that unauthenticated users can access. At a minimum it should contain a button or link that prompts the user to login. The URL of this link should be dynamically set to the value of `__config__.authUrl`. An example client template might look something like so:
```html
<a href="{{authUrl}}">Login with Facebook</a>
```
While the `login.html` page could just be a simple splash page with a login button, it could also be an entire single page app unto itself. For example an eCommerce app might be accessible to anonymous visitors to shop, but the checkout flow requires logging in.
* In your Gruntfile be sure that `login.html` is in the set of files to deploy and that the `sim` task includes a `login` attribute.
```js
aerobatic: {
  deploy: {
    src: ['*.html', 'dist/*.*'],
  },
  sim: {
    index: 'index.html',
    login: 'login.html',
    protocol: 'http', // Or https if SSL is enabled
    port: 3000
  }
}
```
* After the OAuth flow completes Aerobatic will automatically render your `index.html` page. Your JavaScript can access information about the logged in user via the `__config__.user` variable.
* Within your index.html file you'll likely want a logout button. The href for this link should be bound to `__config__.logoutUrl`.
```html
<a href="{{logoutUrl}}">Logout</a>
```

##Making API Calls
Once authenticated you can make calls to the provider's API on behalf of the user. These calls can be made directly from the browser using CORS or you can proxy the calls via the [Aerobatic API proxy]('/docs/backend-integration'). In order to make calls from the browser you will need to pass the access token to the API.

Fetching popular photos from Instagram directly:
```js
$.ajax({
  url: "https://api.instagram.com/v1/media/popular",
  dataType: "jsonp",
  data: {
    access_token: __config__.user.accessToken
  },
  success: function(result) {
  }
})
```

Since the Instagram API doesn't support CORS, any non GET requests would need to be routed through the proxy. So for example liking a post:
```js
$.ajax({
  url: "/proxy",
  dataType: "json",
  data: {
    url: encodeURIComponent("https://api.instagram.com/v1/media/{media-id}/likes?access_token=" + __config__.user.accessToken)
  },
  success: function(result) {
  }
})
```

##Implementation Details
As mentioned the Aerobatic OAuth flow utilizes the web server approach. Once authenticated a session cookie is issued which uniquely identifies the logged in user. Refreshing the browser or leaving the page and returning will remember the user's logged in status. If a user attempts to hit an authenticated URL, they will be redirected to the `login.html` page. This way a single page load does not serve both anonymous and authenticated users providing a clean separation between the two. This approach builds upon the tried and tested approach to managing authentication in a web app. However once a user is authenticated, then the entire app experience takes place within the context of the single `index.html` page.

## Additional Reading
The Salesforce Contacts sample app is a working implementation that you can view the source on GitHub: https://github.com/aerobatic/sfcontacts
The blog post [Securing an HTML5 Salesforce Connected App](http://www.aerobatic.io/blog/2014/07/19/securing-an-html5-salesforce-connected-app) provides additional detail on how OAuth is used with Aerobatic.

### Appendix - Provider Links

OAuth Provider  | Create App Page
------------- | -------------
Salesforce | https://na17.salesforce.com/app/mgmt/forceconnectedapps/forceAppEdit.apexp
Twitter | https://apps.twitter.com/app/new
Google | https://console.developers.google.com/project
Facebook | https://developers.facebook.com/
GitHub | https://github.com/settings/applications
Yammer | https://www.yammer.com/client_applications
Instagram | http://instagram.com/developer/register/
