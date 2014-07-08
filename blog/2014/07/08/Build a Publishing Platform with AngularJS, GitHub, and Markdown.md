Here at Aerobatic, we're all about single page applications. Our marketing website,
www.aerobatic.io, is itself an AngularJS single page app hosted on our
own Aerobatic platform. Of course any proper marketing site requires a blog.
Rather than spin up a separate WordPress at blog.aerobatic.io, I really want to deliver
all content via the single page app. This blog post walks through
the design and implementation of our blog platform. Hopefully it will provide
some valuable insights into building a content publishing platform on a single page
HTML web app. Let's get to it!

### Where to Store Content
Authoring content in Markdown is an obvious starting point, but I need a place
to store all the content. I could include .md files alongside the app code, but
that would entail deploying the app every time we want to publish content. While
Aerobatic makes deployments super easy, deployments ought to be reserved for code
changes only. Another option is to still spin up a WordPress
blog and load the content into our Angular app via the [WordPress JSON API](http://wordpress.org/plugins/json-api/other_notes/).
For an existing WordPress blog this would be a good solution, but since
I'm starting a blog from scratch (this being the inaugural post, meta right?), I'd
like something lighter weight. At Aerobatic we're already heavily reliant on
GitHub; it's where all our [sample apps](#!/gallery) and
[Grunt plugin](https://www.npmjs.org/package/grunt-aerobatic) lives, so storing
blog posts and documentation articles in a GitHub repo is a natural fit. Plus it has
the added benefit of commit history, tracking authorship, and all the version control
capabilities of git. The GitHub website even renders Markdown as HTML, so the
formatted content can be viewed there as well.

### Markdown Repo Structure
For simplicity and clarity, I've decided to structure the folder hierarchy
in the repo to mimic the app's URL structure. I'll adhere to the common convention
of representing the publish date in the URL in the form: `yyyy/mm/dd/title`.
This way I can render a list of links to all blog posts for a particular month if
the day and title are omitted, i.e. `2014/07`. Jakob Nielsen refers to this as [hackable urls](http://www.nngroup.com/articles/url-as-ui/).

```
└── blog
    └── 2014
        ├── 07
        │   └── 08
        │       └── Build a Publishing Platform with AngularJS, Github, and Markdown.md
        └── 08
```
As you can see the title of the Markdown file is the full human readable
title of the blog post itself with a `.md` extension. Once we have everything
wired up, publishing a new blog post is a simple matter of committing a
new Markdown file to the appropriate date folder and pushing to master.

### Retrieving the Blog Index
Before we load the content of actual blog posts, I need a way to build an index
of all blog posts to power the data driven app navigation. Here I use the git concept
of [Trees](http://git-scm.com/book/en/Git-Internals-Git-Objects#Tree-Objects)
exposed through the [GitHub API](https://developer.github.com/v3/git/trees/#get-a-tree-recursively).
I specify the `recursive` parameter since our Markdown files are nested
beneath the dated sub-folder hierarchy.

```
GET https://api.github.com/repos/:owner/:repo/git/trees/:branch?recursive=1
```
To keep things simple, I'm assuming the `master` branch has the content
to render on the site, but you could use any branch name. Here is the actual URL
I'll invoke for the Aerobatic site:

```
https://api.github.com/repos/aerobatic/markdown-content/git/trees/master?recursive=1
```

The JSON response is an object with an attribute `tree` containing a flattened list
of `tree` and `blob` objects. We only care about the objects of type `blob`
corresponding to our Markdown files; the objects of type `tree` can be tossed out.
Here's an abbreviated excerpt of the API response:

```js
{
sha: "master",
url: "https://api.github.com/repos/aerobatic/markdown-content/git/trees/master",
tree: [
    {
      mode: "040000",
      type: "tree",
      sha: "1f3e478b8561093679c3f0b17347c0fb407c703a",
      path: "blog/2014/07/05",
      url: "https://api.github.com/repos/aerobatic/markdown-content/git/trees/1f3e478b8561093679c3f0b17347c0fb407c703a"
    },
    {
      mode: "100644",
      type: "blob",
      sha: "0b14666acdd0a0b621207f6bd4fb971999e3b57e",
      path: "blog/2014/07/05/Build a Blog Platform with AngularJS and GitHub.md",
      size: 2009,
      url: "https://api.github.com/repos/aerobatic/markdown-content/git/blobs/0b14666acdd0a0b621207f6bd4fb971999e3b57e"
    }
  ]
}
```

The GitHub API has both [CORS](https://developer.github.com/v3/#cross-origin-resource-sharing)
and [JSON-P](https://developer.github.com/v3/#cross-origin-resource-sharing) support,
so making an Ajax call from JavaScript in the browser is no problem. However be
aware that GitHub [rate limits](https://developer.github.com/v3/#rate-limiting)
unauthenticted API calls to only 60 requests per hour. Of course I'm hoping this post goes
viral on HackerNews and Twitter and blows right through 60 rph. Authenticated
API requests have much higher limits, but that would require authenticating
blog visitors which certainly is a no-go. Since this blog is running on the
Aerobatic platform, I'm going to leverage the built-in [API proxy](#!/docs/backend-integration)
and specify parameters to force the JSON response from GitHub to be cached in the Aerobatic
cloud. That way large numbers of blog visitors can be served whilst remaining within
the rate limit. With the actual API endpoint wrapped in the proxy, here's the
actual URL that will get invoked from the app:

```
"/proxy?url=https%3A%2F%2Fapi.github.com%2Frepos%2Faerobatic%2Fmarkdown-content%2Fgit%2Ftrees%2Fmaster%3Frecursive%3D1
  &cache=1&ttl=600"
```
Here I'm instructing the Aerobatic proxy to cache the results for 10 minutes (600 seconds).
You could certainly leverage your own caching proxy or use a 3rd party
caching proxy service.

In order to make the blog title URL friendly, I'm going to use the `slugify`
function from [underscore.string](https://github.com/epeli/underscore.string):

```js
_.slugify("Build a Publishing Platform with AngularJS, GitHub, and Markdown")
  => 'build-a-publishing-platform-with-angularjs-github-and-markdown';
```

The final friendly URL for this blog post will be:
```
http://www.aerobatic.io/#!/blog/2014/07/08/build-a-publishing-platform-with-angularjs-github-and-markdown
```
Our app will hold a reference to
the blog index array in memory and use it to data bind to navigational
menus. More on that later.

### Retrieving Content
Now that I have an index of posts and their paths, separate API
requests are made to get the actual content on demand. The GitHub API does
have a [Blob endpoint](https://developer.github.com/v3/git/blobs/#get-a-blob)
that I could pass the `sha` from the Tree call to retrieve the raw content
of the blob. However in my testing I found that the content always came back
base64 encoded. While it's certainly possible to decode base64 strings in the
browser, there is another approach. Every raw source file on GitHub is
directly accessible via a special raw URL that returns the original source code as
plain text. For example, here's raw source URL for the blog post you are reading:

```
https://raw.githubusercontent.com/aerobatic/markdown-content/master/blog/2014/07/08/Build%20a%20Publishing%20Platform%20with%20AngularJS%20and%20Github.md
```
The downside of this approach is it isn't really an API at all, so it isn't CORS
enabled and therefore can't be requested directly from the browser. But this
isn't a problem with the Aerobatic API proxy.

The final issue to address is converting the Markdown to HTML to inject into the DOM.
There are JavaScript libraries like [markdown-js](https://github.com/evilstreak/markdown-js)
that can perform this conversion in the browser, but that adds an additional
client dependency and could potentially slow down the time before the content
is displayed to the visitor. Fortunately the Aerobatic proxy has an additional
transform capability that will perform the conversion in the cloud and send
the final HTML rather than Markdown back to the browser. This is done by simply
tacking on `&transform=markdown` to the proxy URL. Here's the link to this post
with the transform applied: http://www.aerobatic.io/proxy.

It's worth noting that for performance reasons transforms are applied before
writing to the cache, so subsequent requests served from the cache are as
fast as possible.

### Wiring it up with AngularJS
I've chosen to encapsulate all the logic for loading the index and individual
content blobs in an Angular service called simply `content`. Because the content
index is used to power the app navigation, it should be loaded immediately
upon app startup. In Angular this corresponds to the `run` event.

```js
angular.module('aerobatic-io').run(function (content) {
  content.initialize();
});
```
The `initialize` function invokes the GitHub API, parses the JSON into a
list of custom blogPost objects and stores it in the `$rootScope`.

#### Content Service
The complete code for the service is available [here](https://github.com/aerobatic/aerobatic-io/blob/master/js/services/content.js).
```js
var contentIndexDeferred = $q.defer();
return {
  initialize: function() {
    $log.info("Loading content index from GitHub");
    // Go fetch the GitHub tree with references to our Markdown content blobs
    var apiUrl = markdownRepo + '/git/trees/master?recursive=1';

    $http.get('/proxy?url=' + encodeURIComponent(apiUrl) + '&cache=1&ttl=600').success(function(data) {
      var contentIndex = buildIndexFromGitTree(data.tree);
      contentIndexDeferred.resolve(contentIndex);
    }).error(function(err) {
      contentIndexDeferred.reject(err);
      $log.error("Error initializing content index", err);
    });
  },
  contentIndex: function() {
    return contentIndexDeferred.promise;
  },
  load: function(object) {
    // omitted here
  }
};
```
Because the API call is asynchronous, the `intitialize` function creates
a `deferred` object to hold a reference to the operation. The deferred gets
resolved once the API call completes. The service exposes a `contentIndex` function
that returns a reference to the deferred's promise.

The `buildIndexFromGitTree` function analyzes the response to the tree API call
and builds up an index object with for now has a list of blog posts. My plan is
to extend this to support other types of content like documentation and
knowledge base articles. The date attribute is constructed by parsing out the
`yyyy/mm/dd` portion of the path. The underscore.string `_.slugify` function is
used to transform the name of the .md file to a URL friendly representation.
Finally the `blogPosts` are sorted in reverse chronological order.

```js
function buildIndexFromGitTree(tree) {
  var index = {
    blogPosts: []
  };

  _.each(tree, function(node) {
    if (node.type === 'blob') {
      // Value of path is in format 'blog/yyyy/mm/dd/title.md'
      var path = node.path.split('/');
      if (path[0] === 'blog') {
        // Strip off the .md extension
        var title = _.strLeftBack(path[4], '.');

        // Use underscore.string to slugify the title
        var titleSlug = _.slugify(title);

        index.blogPosts.push({
          // Build a JS date from '2014/07/05'
          date: new Date(parseInt(path[1]), parseInt(path[2]) - 1, parseInt(path[3])),
          title: title,
          sha: node.sha,
          gitPath: node.path,
          titleSlug: titleSlug,
          // Use underscore.string slugify function to get a URL safe
          // reprensentation of the title
          urlPath: '/' + path.slice(0, 4).concat(titleSlug).join('/')
        });
      }
    }
  });

  // Sort the blogPosts in reverse chronological order
  index.blogPosts = _.sortBy(index.blogPosts, 'date').reverse();
  return index;
}
```

The last bit of the service is a `load` function which is used to fetch an
individual piece of content which will be automatically transformed to HTML
by the Aerobatic API proxy.

```js
load: function(object) {
  // Rather than the GitHub API, just grab the raw source.
  var apiUrl = 'https://raw.githubusercontent.com/aerobatic/markdown-content/master/' + encodeURIComponent(object.gitPath);

  var deferred = $q.defer();

  var proxyUrl = '/proxy?url=' + encodeURIComponent(apiUrl);

  // Set to cache for 1 hour
  proxyUrl += '&cache=1&ttl=' + (60 * 60);

  // Specify that the API response should pass through the markdown transform
  proxyUrl += '&transform=markdown';

  $http.get(proxyUrl).success(function(data) {
    deferred.resolve(data);
  }).error(function(err) {
    deferred.reject(err);
  });

  // Return a promise to the caller
  return deferred.promise;
}
```

Once again a promise is returned to the caller which in this case is
[blogCtrl.js](https://github.com/aerobatic/aerobatic-io/blob/master/js/controllers/blogCtrl.js).
But before looking at the controller, we need to register routes with
the [angular router](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider).
All the route parameters are optional in order to handle `/blog`, `/blog/yyyy/[mm]/[dd]`, and
of course the full route for a specific blog post `/blog/yyyy/mm/dd/title`.

```js
angular.module('aerobatic-io').config(function($routeProvider, $locationProvider) {
  // Use the bang prefix for Google ajax crawlability
  // https://developers.google.com/webmasters/ajax-crawling/docs/specification?csw=1
  $locationProvider.hashPrefix('!');

  $routeProvider.when('/', { template: JST['partials/index']() })
    .when('/blog/:year?/:month?/:day?/:title?', {
      template: 'partials/blog',
      controller: 'BlogCtrl'
    })
    .otherwise({ redirectTo: '/' });
});
```

Because a blog post could be the first view rendered by the app, I can't assume
the contentIndex has already loaded, so I defer the controller logic until the
`contentIndex()` promise has been resolved. If it has already resolved, then the
callback will execute immediately.

The `blogCtrl` examines the `$routeParams` and determines whether an index of
blog posts or the content of a blog post should be rendered. If the `$routeParams`
is empty then I automatically change the `$location.path` to be the most
recent blog post. This allows for a convenient permalink http://www.aerobatic.io/#!/blog
that will always lead to the latest post. If the `$routeParams` has all 4 parts,
then I locate the blog post whose `urlPath` matches `$location.path()` and pass
it to the `load` function of the content service.

Finally before assigning the resulting HTML to the `$scope` to be bound to the
view, it's critical to indicate to Angular that the content is safe to be
treated as HTML with the [$sce service](https://docs.angularjs.org/api/ng/service/$sce).

```js
angular.module('controllers').controller('BlogCtrl', function($scope, $location, $routeParams, $sce, content) {
  content.contentIndex().then(function(contentIndex) {
    var routeKeys = _.keys($routeParams);
    if (routeKeys.length === 0) {
      if (contentIndex.blogPosts.length === 0) {
        $log.debug("No blog posts");
        return $location.path('/');
      }
      else {
        // If this is /blog URL with no extra parameters, change
        // the path to the most recent post.
        return $location.path(_.first(contentIndex.blogPosts).urlPath);
      }
    }
    // There must not be a title, so render the blog index
    else if (routeKeys.length < 4) {
      var datePrefix = _.compact([$routeParams.year, $routeParams.month, $routeParams.day]).join('/');
      $scope.indexPosts = _.filter(contentIndex.blogPosts, function(post) {
        return _.startsWith(post.gitPath, 'blog/' + datePrefix);
      });
      return;
    }
    // If the route has all 4 parts yyyy/mm/dd/title then render the actual post
    else {
      // Load the content and write it to the page
      var blogPost = _.find(contentIndex.blogPosts, {urlPath: $location.path()});
      if (!blogPost)
        return $location.path('404');

      $scope.blogPost = blogPost;
      content.load(blogPost).then(function(content) {
        $scope.content = $sce.trustAsHtml(content);
      });
    }
  });
});
```

### Wrapping Up
That pretty much takes care of it. Our single page marketing app now has a
functional blog (you're looking at it). Publishing new content is as easy as
authoring Markdown files and pushing them to the [aerobatic/markdown-content](https://github.com/aerobatic/markdown-content)
GitHub repo.

I've glossed over some other important
considerations including SEO which requires some special measures in single
page apps; I'll cover that in detail in a future post. RSS support is another
feature that I still need to tackle. But in the meantime I look forward to
sharing more about Aerobatic and how it can help supercharge the delivery of
single page HTML5 apps.
