Getting search crawlers to index content delivered asynchronously to the page has traditionally been a major drawback to content-driven single page applications. Fortunately that is starting to change. In May [Google announced](http://googlewebmastercentral.blogspot.com/2014/05/understanding-web-pages-better.html) that they will now attempt to read content that is only accessible via JavaScript. It's still early days, so results are inconsistent. If you find that your app's content is not getting indexed correctly, Aerobatic provides support for rendering HTML snapshots to search crawlers.

## Snapshot Technique
There are two basic styles for AJAX urls: HTML5 pushState and hashbangs. The pushState approach is highly recommended, both for aesthetic reasons and because both [Google](https://www.seroundtable.com/google-ajax-pushstate-vs-hashbang-16464.html) and Bing prefer it. The grunt-aerobatic package includes a snapshot task which will load a URL in a headless browser and capture the full state of the DOM once all content has loaded.

### Fragment Meta Tag
In the head of your `index.html` page, make sure you declare the following tag informing the search bot that you are opting into the AJAX crawling scheme:
```html
<meta name="fragment" content="!">
```

### Framework Support
All of the major JavaScript MVC framework support HTML5 pushState in their routing modules. For Angular apps, you configure this with the [$locationProvider](https://docs.angularjs.org/api/ng/provider/$locationProvider):

```js
app.config(function($routeProvider, $locationProvider) {
  $locationProvider.html5Mode(true);  
});
```
Use plain anchor tags, rather than say button elements, for any navigational controls that you want the search bot to follow. With CSS, you can always style anchor elements to look like buttons. Inspecting the DOM, these anchors should look similar to this:

```html
<a href="/new-view">Load New View</a>
```
## Taking Snapshots
To take a snapshot of your pages, you can use the `snapshot` Grunt task that comes with the [grunt-aerobatic](https://www.npmjs.org/package/grunt-aerobatic) package.

```bash
grunt aerobatic:snapshot url=http://your_app.aerobaticapp.com/page-name
```
In your `Gruntfile.js` the task is configured like shown below. The only option is the `timeout` which specifies the number of ms to wait to allow all content to be appended to the DOM. The default is 3 seconds.
```js
aerobatic: {
  snapshot: {
    all: { timeout: 2000 }
  }
}
```

The task uses [PhantomJS](http://phantomjs.org/) to load the URL in a headless browser and capture the state of the DOM after the specified timeout. The resulting HTML string is uploaded to the Aerobatic cloud.

## Rendering Snapshots
When the Googlebot crawls your app, it will detect the meta tag and re-request your app with a modified "ugly URL". For example if the crawler hit `http://your_app.aerobaticapp.com/page-name`, it would make a new request to `http://your_app.aerobaticapp.com?_escaped_fragement_=page-name`. The Aerobatic platform will take care of responding with the stored snapshot corresponding to `page-name`. In the search results, Google will translate the link back to the "pretty url" so visitors will arrive from search results via the `http://your_app.aerobaticapp.com/page-name` URL.

You can see the snapshot for this article at this URL: http://www.aerobatic.io?_escaped_fragment_=docs%2Fseo. Because the snapshot is just static HTML, the response time back to the search crawler will be very fast.

## Sitemap.xml
Even if your single page app is crawlable, it's a good idea to maintain a sitemap. This should contain the absolute URLs that match the `href` attributes in the DOM. Your `sitemap.xml` file should reside in the root of your application source directory and look something like so:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
   <url>
      <loc>http://your_app.aerobaticapp.com/</loc>
      <changefreq>weekly</changefreq>
      <priority>0.8</priority>
   </url>
   <url>
      <loc>http://your_app.aerobaticapp.com/page-name</loc>
      <changefreq>weekly</changefreq>
      <priority>0.8</priority>
   </url>
</urlset>
```

Aerobatic makes the sitemap available at `http://your_app.aerobaticapp.com/sitemap.xml` in accordance with the default location that search crawlers look for it. You should also register this URL in the [Google Webmaster tools](https://www.google.com/webmasters/tools).

## Page Titles
The `<title>` element is considered the most valuable piece of metadata when it comes to SEO. Your single page application should dynamically set the title whenever the view changes. When you take the snapshot of the URL, the dynamic title will be reflected in the captured HTML and be reflected in the search engine's index.

Here's a simple way to set the page title in Angular:
```js
$document[0].title = dynamicPageTitle;
```
