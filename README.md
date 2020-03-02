## Overview

The faceted search browsers are powered by [FacetView2](https://github.com/tetherless-world/facetview2) - a pure javascript frontend for ElasticSearch search indices that let you easily embed a faceted browse front end into any web page.
This is now just copied locally under this repo.

To configure a working faceted browser you need:
1. A running instance of ElasticSearch with a populated index - in this case experts.colorado.edu/es
2. A webpage with references to facetview2 scripts and an embedded configuration

That's it!

You can find out more about ElasticSearch at [https://www.elastic.co/products/elasticsearch](https://www.elastic.co/products/elasticsearch)


### Publications page that already works
You should be able to copy this repo to your webserver, then hit the publications.html page. It should query to our running elasticsearch instance for data


### Setting up the faceted browser from scratch

Embedding a faceted browser in a webpage is very simple.

Add the following code to your web page (paths will be different for some deployment environments):
```html
    <script type="text/javascript" src="facetview2/vendor/jquery/1.7.1/jquery-1.7.1.min.js"></script>
    <link rel="stylesheet" href="facetview2/vendor/bootstrap/css/bootstrap.min.css">
    <script type="text/javascript" src="facetview2/vendor/bootstrap/js/bootstrap.min.js"></script>
    <link rel="stylesheet" href="facetview2/vendor/jquery-ui-1.8.18.custom/jquery-ui-1.8.18.custom.css">
    <script type="text/javascript" src="facetview2/vendor/jquery-ui-1.8.18.custom/jquery-ui-1.8.18.custom.min.js"></script>
    <script type="text/javascript" src="facetview2/es.js"></script>
    <script type="text/javascript" src="facetview2/bootstrap2.facetview.theme.js"></script>
    <script type="text/javascript" src="facetview2/jquery.facetview2.js"></script>
    <link rel="stylesheet" href="facetview2/css/facetview.css">
    <link rel="stylesheet" href="browsers.css">
```

Then add a script somewhere to your page that actually calls and sets up the facetview on a particular page element: 
```js
<script type="text/javascript">
jQuery(document).ready(function($) {
  $('.facet-view-simple').facetview({
    search_url: 'http://localhost:9200/myindex/type/_search',
    facets: [
        {'field': 'publisher.exact', 'size': 100, 'order':'term', 'display': 'Publisher'},
        {'field': 'author.name.exact', 'display': 'author'},
        {'field': 'year.exact', 'display': 'year'}
    ],
  });
});
</script>
```

Then add a ``div`` with the HTML class referenced in the script to the HTML body.
```
</head>
    <body>
        <div class="facet-view-simple"></div>
    </body>
</html>
```

That should be it!

See the webpages in /html for examples of more complex facetview2 configurations and for how to use JS or templates to specify the HTML shown for result set entries.
