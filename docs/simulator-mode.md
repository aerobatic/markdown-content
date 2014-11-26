

## Simulator Mode
The simulator mode is a unique feature of Aerobatic that provides a fully integrated development environment. Typically when building static apps you work off of a localhost URL. This works well for simple apps, but there are scenarios where it breaks down. For example when registering with an OAuth provider you have to specify the callback URL that the provider will redirect to and this URL has to match the origin of the request from your actual app, not `localhost`. Another potential scenario is making a CORS request to a remote domain that has rules on what domains are allowed. By developing and testing on your production domain, you eliminate these and other potential environment discrepancies and uncover integration issues early on.

You start the simulator with the simple command `grunt aerobatic:sim --open`. This uploads your `index.html` file to the Aerobatic cloud platform where it is stored in a private cache that is specific to your individual Aerobatic userId. This way you are still fully isolated from live traffic or any other developers. The sim task watches the local `index.html` file and will automatically upload a new copy whenever it changes.

 The port number can be overridden in the grunt config if you choose. The URL that you point to during development is the same as your production app URL but with some additional query parameters:
 ```
http:<your_app_name>.aerobaticapp.com?sim=1&user=<your_user_is>&port=3000&reload=1
```
Rather than have to type this all in, you can simply pass the `--open` command line option which will launch a browser to the right fully formed URL.

When Aerobatic sees the `sim=1` in the URL it knows that the request should be treated differently than a normal request. The cached index.html for your userId is grabbed from cache and undergoes the same basic processing as a regular request by removing any `data-build-type="release"` elements and fixing any referenced assets to http://localhost:3000 (the port number can be overridden in the Gruntfile). Additionally a banner overlay is injected into `index.html` that appears in the top right corner of the page which lets you easily see at a glance that you are in simulator mode.