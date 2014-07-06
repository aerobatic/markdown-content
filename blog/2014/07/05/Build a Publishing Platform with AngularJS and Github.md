Here at Aerobatic, we're all about single page applications. Our website www.aerobatic.io
is itself an AngularJS single page application hosted on the Aerobatic platform.
Any proper marketing site needs a blog. Rather than spin up a separate WordPress
at blog.aerobatic.io, I really want to keep all our content part of our single
page website. That means we need a place to store our blog content. One option
would be to still have a WordPress blog, but load the content into our Angular app
via the [WordPress JSON API](http://wordpress.org/plugins/json-api/other_notes/).
If you have an existing WordPress blog this would be a good solution, but since
I'm starting a blog from scratch (this being the inaugural post, meta right?), I
think we can do something lighter weight. We're already heavy users of GitHub as
that's where all our [sample apps](#!/gallery) and [Grunt plugin](https://www.npmjs.org/package/grunt-aerobatic) lives, so why not store
our blog posts as Markdown files in a GitHub repo.

### Markdown Repo Structure
For simplicity and transparency, I've decided to structure the folder hierarchy
in the repo to mimic the site's URL structure. Additionally we can adhere to the
convention of representing the publish date in the URL in the form: `yyyy/mm/dd/title`.
This way we can render a list of links to all blog posts for a particular month if
the day and title are omitted, i.e. `2014/07`. Jakob Nielsen called this structure
as [hackable urls](http://www.nngroup.com/articles/url-as-ui/).

```
└── blog
    └── 2014
        ├── 07
        │   └── 05
        │       └── Build a Publishing Platform with AngularJS and Github.md
        └── 08
```
As you can see the title of the Markdown file is the full human readable
title of the blog post itself with a `.md` extension. Once we are wired up
to the site, publishing a new blog post is a simple matter of committing a
new Markdown file in the appropriate date folder and pushing to GitHub.
