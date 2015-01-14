<meta id="blogShortUrl" value="http://bit.ly/1rb6RTo">
<meta id="blogAuthorBio" value="David Von Lehman is co-founder of Aerobatic. He oftentimes has visions of JavaScript, cloud platforms, and single page applications dancing in his head. Reach him at @davidvlsea or david@aerobatic.com">

We're happy to announce support for [Google OAuth2](https://developers.google.com/accounts/docs/OAuth2) in your Aerobatic apps. Not only can you authenticate users with their Google account, but also integrate with any number of the dizzying assortment of Google REST APIs (I count over 60 in the [API Explorer](https://developers.google.com/apis-explorer/?csw=1#p/)) on behalf of the logged in user. This opens the door to all sorts of interesting app integration possibilities with the likes of Gmail, Calendar, Drive, even BigQuery.

While you can implement the OAuth2 flow [entirely in client script](https://developers.google.com/accounts/docs/OAuth2UserAgent), Aerobatic makes it much simpler by taking care of handling it from the server-side on your app's behalf so your client code doesn't need to worry about detecting login state, redirecting to a login screen, etc. In fact your app is only loaded to the browser after the user has successfully authenticated.

### Creating New Google App
First you'll need to login to the [Google Developer Console](https://console.developers.google.com) and create a new project. After you enter the app name, there's an extra step that isn't at all obvious which will trip you up later. Click on the __Consent screen__ link in the left nav and ensure an email address is selected in the dropdown. Also ensure the __Product Name__ is identical to the __Project Name__ that you used to create the project. Don't ask me why this matters or why they are two separate fields to begin with, but it may result in an `invalid_client` OAuth error. See this [StackOverflow answer](https://developers.google.com/accounts/docs/OAuth2UserAgent) for more details. I shudder to think how much time would have been wasted on this had SO not saved the day once again.

Now click on the __Credentials__ link in the left nav and make the following settings:

Field  | Value
------------- | -------------
Redirect URI | https://[yourapp].aerobaticapp.com/auth/callback
Javascript Origin | https://[yourapp].aerobaticapp.com/

Back in the Aerobatic portal visit the Settings screen for your app, click the OAuth option and select __Google__ in the provider dropdown. Copy and paste your __ClientID__ and __Client Secret__ from the developer console into the respective text fields. Next specify which [scopes](https://developers.google.com/accounts/docs/OAuth2Login#scope-param) you want to ask user consent for. At a minimum you'll need to specify a ["profile" scope](https://developers.google.com/+/api/oauth#profile) which simply grants the app access to the logged in user's basic profile information such as name and profile photo. You can ask for additional API scopes that will allow your app to make API calls on behalf of the logged in user to access their Gmail or Calendar data. Here's a [partial reference](https://developers.google.com/gdata/faq#AuthScopes) of the various API scopes, but it's not comprehensive. The Google Developer documentation is expansive, you're best bet is to search for "Google OAuth {API name} scope".

<img class="img-responsive" src="https://s3-us-west-2.amazonaws.com/aerobatic-media/aerobatic-portal-oauth-google.png">

The remaining step is to implement a `login.html` page that renders a link pointing to `/auth` which will automatically kick off the OAuth login flow. Thanks the Aerobatic dev simulator you can run a sandboxed instance of your app hosted on the production URL so all the OAuth just works even while coding locally. The [authentication documentation](/docs/authentication) has more details on the actual code implementation.

### Making API Calls
Once the user is authenticated, the `__config__.user` object will expose a `accessToken` attribute which needs to be passed along in API calls in the `Authorization` header. You could choose to include the [Google API Client Library for JavaScript](https://developers.google.com/api-client-library/javascript/) in your app which provides an abstraction layer atop the REST APIs. However since Aerobatic has already taken care of the OAuth piece, I find it more straightforward to invoke the REST APIs directly which are intuitive and [well documented](https://developers.google.com/apis-explorer/). Plus you are probably already using a JavaScript library such as Angular with strong support for invoking REST services and returning promises, so the Google client library adds a bunch of redundant functionality, not to mention larger page weight.

Here's an example call to retrieve the list of calendars using the [Calendar API](https://developers.google.com/google-apps/calendar/):

```js
$.ajax({
  url: 'https://www.googleapis.com/calendar/v3/users/me/calendarList',
  headers: {
    'Authorization': 'Bearer ' + __config__.user.accessToken,
  }
});
```

### Custom Google Apps Hosted Domain
That's great that you can require app users to authenticate with Google, but still anybody with a valid Google account can login. What if your app is intended to only be used by employees of your company? If you are running Google Apps for Work with a custom domain, you're in luck! There is an optional `hd` option that can be passed specifying a hosted domain restricting access to user's whose account belongs to that domain. This doesn't receive much fanfare in the developer documentation, but it's a very powerful enhancement that now makes it possible to run internal facing apps on the public cloud. In order to enable this in Aerobatic, simply enter your domain in the form "mycompany.com" in the __Hosted Domain__ text field, and viola, you're app is locked down to only members of your company.

### Summary
Google is in the midst of unifying all their various services under the Google Cloud Platform umbrella. Fortunately for web developers, all these APIs are CORS enabled and callable directly from the browser which opens up a lot of exciting client app possibilities - both public and internal facing. The Aerobatic OAuth module automates the entire login flow so you can focus on the fun stuff.

Happy Coding!
