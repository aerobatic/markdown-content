Here at Aerobatic, we're all about single page applications. Our marketing website,
www.aerobatic.io, is itself an AngularJS single page app hosted on our
own Aerobatic platform. Obviously any proper marketing site needs a blog. Rather
than spin up a separate WordPress at blog.aerobatic.io, I really want to keep
all our content part of our single page website.

### Where to Store Content
Authoring content in Markdown is an obvious starting point, but I need a place
to store all the content. I could include .md files alongside the app code, but
that would entail deploying the app every time we want to publish content. While
Aerobatic makes deployments super easy, it's still not recommended as deployments
should be reserved for code changes. Another option is to still spin up a WordPress
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
  &cache=1&key=markdown-content-index&ttl=600"
```
Here I'm commanding the Aerobatic proxy to cache the results under the key `aerobatic-markdown-content`
for 10 minutes. You could certainly leverage your own caching proxy or use a 3rd party
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
with the transform applied.

It's worth noting that for performance transforms are applied before
writing to the cache.





### Wiring it up with AngularJS
