# SEO Snapshots

Getting search crawlers to index content delivered asynchronously to the page has traditionally been a major drawback to content-driven single page applications. Fortunately that is starting to change. In May 2014 [Google announced](http://googlewebmastercentral.blogspot.com/2014/05/understanding-web-pages-better.html) that they will now attempt to read content that is only accessible via JavaScript. It's still early days, so results are inconsistent. If you find that your app's content is not getting indexed correctly, Aerobatic provides support for rendering HTML snapshots to search crawlers.

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
## Snapbot
Aerobatic has a backend service known as Snapbot, which will crawl your app regularly, starting with the index page. Any anchor element it finds with an href attribute will be added to the set of URLs to snapshot. The actual snapshot process entails loading your page in a headless browser (we use [SlimerJS](http://slimerjs.org/)). Once the initial index page is loaded, the process waits 5 seconds to give time for all JavaScript and CSS to load, any API calls to complete, and DOM rendering to finish. At that time the state of the DOM is captured and written to a static HTML file on S3. 

## Rendering Snapshots
When the Googlebot crawls your app, it will detect the meta tag and re-request your app with a modified "ugly URL". For example if the crawler hit `http://your_app.aerobaticapp.com/page-name`, it would make a new request to `http://your_app.aerobaticapp.com/page_name?_escaped_fragement_=`. Aerobatic detects the presence of the `_escaped_fragment_` parameter and returns the pre-rendered snapshot instead of the actual page. In search results, Google translates the link back to the "pretty url" so visitors to your app will have the correct referer: `http://your_app.aerobaticapp.com/page-name`.

You can see the snapshot for this article at this URL: http://www.aerobatic.com/docs/seo?_escaped_fragment_=. Because the snapshot is just static HTML, the response time back to the search crawler will be very fast.

It's worth mentinoning that this technique is completely sanctioned by Google and does not represent any sort of "black hat" cloaking.

## Sitemap.xml
Aerobatic will automatically generate a sitemap on-demand when a request arrives for `/sitemap.xml`. The URLs included are the ones that were discovered as part of the most recent crawl cycle by Snapbot. If you want to have full control over the sitemap yourself, you can simply include a `sitemap.xml` file in the root of your deployments. Note that sitemaps must have absolute URLs. 

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

You should also register your sitemap URL in the [Google Webmaster tools](https://www.google.com/webmasters/tools).

## Page Titles
The `<title>` element is considered the most valuable piece of metadata when it comes to SEO. Your single page application should dynamically set the title whenever the view changes. When you take the snapshot of the URL, the dynamic title will be reflected in the captured HTML and be reflected in the search engine's index.

Here's a simple way to set the page title in Angular:
```js
$document[0].title = dynamicPageTitle;
```
