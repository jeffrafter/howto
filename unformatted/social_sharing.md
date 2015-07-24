# Social Sharing

Sharing on social platforms is changing constantly. But for the moment you can customize links to share on social fairly easily. In most cases it makes sense to do so without embedding the social specific JavaScript.


## Open Graph tags

Open graph tags are used by both Pinterest and Facebook:


    <meta property="og:type" content="website">
    <meta property="og:title" content="A good title">
    <meta property="og:image" content="http://exmaple.com/sample.jpg">
    <meta property="og:image:width" content="1200">
    <meta property="og:image:height" content="630">
    <meta property="og:description" content="A long desc which is html escaped">
    <meta property="og:url" content="http://example.com">
    <meta property="og:site_name" content="A Friendly name for your site">

For facebook to pre-render your image in the popup, you must include the `og:image:width` and `og:image:height` tags. Ideally your shared images will be 1200 x 630;

More info Open Graph meta tags:

- [https://developers.facebook.com/docs/opengraph/using-objects/](https://developers.facebook.com/docs/opengraph/using-objects/)
- [https://developers.pinterest.com/rich_pins/](https://developers.pinterest.com/rich_pins/)

Use the Facebook Open Graph Debugger for validation (and cache clearing):

- [https://developers.facebook.com/tools/debug](https://developers.facebook.com/tools/debug)

Validate your Pinterest rich pins:

- [https://developers.pinterest.com/rich_pins/validator/](https://developers.pinterest.com/rich_pins/validator/)

## Twitter cards

Twitter has its own card style:

    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:site" content="@yourtwitter">
    <meta name="twitter:creator" content="@yourtwitter">
    <meta name="twitter:title" content="Your Title>
    <meta name="twitter:description" content="Your description, html escaped">
    <meta name="twitter:image" content="http://example.com/sample.jpg">

More info about twitter cards:

- [https://dev.twitter.com/cards/types/summary-large-image](https://dev.twitter.com/cards/types/summary-large-image)


## Linking to share

When passing the parameters via URL you must URL escape the text (it may contain reserved characters and spaces). The following sample uses liquid to render in the parameters. You can use anything:

Pinterest:

    <a class="icon-fallback-text" target="_blank" href="//www.pinterest.com/pin/create/button/?url={{ share_url | url_escape_text }}&media={{ share_image | url_escape_text }}&description={{ share_pinterest_text | url_escape_text }}" title="Share on Pinterest" onclick="window.open(this.href,'_blank', 'width=700, height=300'); return false">
      <i class="fa fa-pinterest"></i>
      <span class="fallback-text">Pinterest</span>
    </a>
    
Twitter:
    
    <a class="icon-fallback-text" target="_blank" href="https://twitter.com/intent/tweet?text={{ share_twitter_text | url_escape_text }}&url={{ share_url | url_escape_text }}&via=snowehome" title="Share on Twitter" onclick="window.open(this.href,'_blank', 'width=585, height=261'); return false">
      <i class="fa fa-twitter"></i>
      <span class="fallback-text">Twitter</span>
    </a>
    
Facebook:
    
    <a class="icon-fallback-text" target="_blank" href="http://www.facebook.com/share.php?u={{ share_url | url_escape_text }}&t={{ share_facebook_text | url_escape_text }}&v=4" title="Share on Facebook"  onclick="window.open(this.href,'_blank', 'width=585, height=368'); return false">
      <i class="fa fa-facebook"></i>
      <span class="fallback-text">Facebook</span>
    </a>

This example also uses social icons which are leveraging font-awesome. You don't have to do that.

    {{ '//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css' | stylesheet_tag }}
