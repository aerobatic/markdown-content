<meta id="blogShortUrl" value="http://bit.ly/1rb6RTo">
<meta id="blogAuthorBio" value="David Von Lehman is co-founder of Aerobatic. He oftentimes has visions of JavaScript, cloud platforms, and single page applications dancing in his head. Follow him @davidvlsea">

Every time I go to implement the ubiquitous social sharing icons I'm struck anew what a time sink it always turns into. All the major social networks now have their proprietary widgets that entail gobs of JavaScript that get injected into your page. All this seems designed more to benefit their ability to track where on the internet their members visit than adding actual value to your own site. On top of the JavaScript bloat, there is the inevitable css hair-pulling exercise required to get all those little icons, each wrapped up in the DOM with funky class names (or even worse, iframes), to line up properly and provide some semblance of design uniformity. Sure there are no shortage of JavaScript plugins to render share icons for every social network under the sun, but that merely abstracts the messiness and page performance impacts. At the end of the day, all I want is to provide a convenient way for our blog visitors to share a link to the post. I'm willing to trade-off little gimmicks like real-time share count in exchange for a simple, lightweight, and transparent implementation.

Fortunately the major networks still provide a simple URL based share mechanism, even if you have to hunt for it in the documentation. With the URL convention in hand, we have all we need to roll our own set of share button that launch the form in a new window.

Here's the relevant docs for the networks I'm interested in:

* [Google Plus](https://developers.google.com/+/web/share/#sharelink)
* [Linked In](https://developer.linkedin.com/documents/share-linkedin)
* [Twitter](https://dev.twitter.com/docs/tweet-button)
* [Reddit](http://www.reddit.com/buttons/)

## Short URLs
Initially I was just using `window.location.href` as the url to share, but I found that LinkedIn was chopping off everything after the `#` symbol in the url. Since this app's Ajax navigation scheme relies on the hashbang convention, this would have prevented deep-linking directly to the blog post view. Also the full URL looked rather ungainly in the Twitter textbox. To work around this I made a small improvement to the Markdown based publishing system which I wrote about in detail in the post: [Build a Publishing Platform with AngularJS, GitHub, and Markdown](/blog/2014/07/08/build-a-publishing-platform-with-angularjs-github-and-markdown). Taking advantage of the fact that raw HTML is supported in Markdown, I just embed a `meta` tag at the top of each .md file with the id `blogShortUrl` that specifies a bit.ly short url for the post. The JavaScript reads the value out of the DOM and passes it along in the share URLs.

```html
<meta id="blogShortUrl" value="http://bit.ly/1rb6RTo" />
```
Now the links are devoid of any problematic characters and they're nice and succinct in the share post, and as a bonus we get the link tracking capabilities of bit.ly.

## Angular Implementation
Here's the full code which is implemented as an Angular directive with a little help from [lodash](http://lodash.com/):
```javascript
angular.module('directives')
.directive('blogShare', function($window, $document, $log) {
  var networks = {
    linkedin: {
      // https://developer.linkedin.com/documents/share-linkedin
      url: "https://www.linkedin.com/shareArticle?mini=true&url={{href}}&title={{title}}&source={{source}}"
    },
    google: {
      // https://developers.google.com/+/web/share/#sharelink
      url: "https://plus.google.com/share?url={{href}}"
    },
    twitter: {
      url: "https://twitter.com/intent/tweet?text={{title}}&url={{href}}&via={{twitterHandle}}"
    },
    reddit: {
      url: "http://www.reddit.com/submit?url={{href}}&title={{title}}"
    },
    hackernews: {
      url: "http://news.ycombinator.com/submitlink?u={{fullHref}}&t={{title}}"
    }
  };

  return {
    restrict: 'C',
    link: function(scope, element, attr) {
      var popupUrl;
      var network = networks[attr.shareNetwork];
      if (!network) {
        $log.error("Invalid blog share network " + attr.shareNetwork);
        return;
      }

      element.bind('click', function() {
        var shortUrl = angular.element(document.querySelector("#blogShortUrl")).attr("value");
        var blogTitle = angular.element(document.querySelector("#blogTitle")).text();

        var params = {
          fullHref: encodeURIComponent($window.location.href),
          href: encodeURIComponent(shortUrl || $window.location.href),
          title: encodeURIComponent(blogTitle),
          twitterHandle: 'aerobaticapp',
          source: encodeURIComponent('Aerobatic Blog')
        };

        _.defaults(network, {
          width: 600,
          height: 600
        });

        var shareUrl = _.template(network.url, params);
        var windowOptions = _.template("menubar=no,location=no,resizable=no,"+
          "scrollbars=no,status=no,width={{width}},height={{height}}," +
          "top=100,left=100", network);

        $window.open(shareUrl, 'sharePopup', windowOptions);
        this.blur();
      });
    }
  };
});
```
The HTML for the buttons is nice and semantic:
```html
<button share-network="linkedin" class="btn blog-share linkedin">
  <i class="icon-linkedin"></i>
</button>
<button share-network="google" class="btn blog-share google">
  <i class="icon-gplus"></i>
</button>
<button share-network="twitter" class="btn blog-share twitter">
  <i class="icon-twitter"></i></button>
<button share-network="reddit" class="btn blog-share reddit">
  <i class="icon-reddit"></i>
</button>
```

Rather than images, the icons are part of a custom icon font bundle I generated on [Fontello](http://fontello.com/).

So there you have it, a widget-free set of sharing buttons that don't require any extra JavaScript libs. As an improvement I plan to hook the button click events up with Google Analytics so the action is recorded. It won't be a perfect measurement since the visitor could never actually submit the share, but hopefully that is the exception rather than the rule. You can see the final product below. By all means, feel free to try them out;)

Happy Coding!
