## Overview

The faceted search browsers are powered by [FacetView2](https://github.com/tetherless-world/facetview2) - a pure javascript frontend for ElasticSearch search indices that let you easily embed a faceted browse front end into any web page.
This is now just copied locally under this repo because the original repositories are no longer maintained.
Additionally, the local code has been modified to work with Elasticsearch versions > 7.

To configure a working faceted browser you need:
1. A running instance of ElasticSearch with a populated index 
   - in this case: https://search-experts-direct-cz3fpq4rlxcbn5z27vzq4mpzaa.us-east-2.es.amazonaws.com/fispubs-v1
2. A webpage with references to facetview2 scripts and an embedded configuration

That's it!

You can find out more about ElasticSearch at [https://www.elastic.co/products/elasticsearch](https://www.elastic.co/products/elasticsearch)

## Using FacetView2

### Example people and publication pages for CU Experts and CU Experts Direct

There are example pages to support people and publication pages.
The .ftl files are for inclusion in CU Experts.
The .html files are for standalone pages.

To install these pages
1. Copy this current repository to the target directory which will serve these assets
2. Copy the publication.html page to the location that will serve them
3. You probably have to adjust the pathing of the css and javascript assets called in the pages.
4. You might have to adjust the search_url variable to point to the Elastic index.
5. The production Elastic index on AWS uses basic http authentication, the username and password below are for public access so they are not a secret.

Next just hit the publications.html page. It should query  our running elasticsearch instance for data

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
    search_url: 'https://search-experts-direct-cz3fpq4rlxcbn5z27vzq4mpzaa.us-east-2.es.amazonaws.com/fispubs-v1/_search',
    username: 'anon',
    password: 'anonyM0us!',
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


### Customisation

FacetView2 has been written to allow extensive customisation within a flexible but constrained page framework.

#### Pre-populating a page with a set of users
You might want to create a page with a certain set of users. This can be tricky of the users aren't organized in a group.
You can use the predefined_filters{} object to do this.
If you want to use a predefined filter, that filter also needs to be a facet, but the facet can be hidden.
Below is an example of how to do this, note the authors.fisID.keyword facet and how the final attribute is hidden:

               predefined_filters : {
                   "authors.fisId.keyword" : ["118372","140077"]
                },
                facets: [
                    {'field': 'mostSpecificType.keyword', 'display': 'Publication Type'},
                    {'field': 'authors.name.keyword', 'size': 20, 'display': 'Author'},
                    {'field': 'authors.fisId.keyword', 'size': 20, 'display': 'FisID', 'hidden': true},
                    {'field': 'publishedIn.name.keyword', 'display': 'Published In'},

So far I also couldn’t add multiple predefined filters, for example adding authors.name.keyword underneath the authors.fisId.keyword didn’t work.
               predefined_filters : {
                   "authors.fisId.keyword" : ["118372","140077"],
                   “authors.name.keyword”: [“Elsborg, Don”]
                },


## Querying the Elasticsearch index directly

It is possible to just query the Elasticsearch index for data and bypass the javascript interface.
Please reach out to the ODA Engineering team for the appropriate Elasticsearch endpoint.
This page won't go into details on how to construct an Elasticsearch query. Details can be seen on the Elasticsearch guide: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-your-data.html
Query examples below follow formats similar to that used by the Facetview software. They consist of filters and aggregations.

### Elasticsearch endpoint on AWS server
# A username and password is required. The public credential is listed below.
    search_url: 'https://search-experts-direct-cz3fpq4rlxcbn5z27vzq4mpzaa.us-east-2.es.amazonaws.com/fispubs-v1/_search',
    username: 'anon',
    password: 'anonyM0us!'


### Elasticsearch publication queries for common identifiers such as department ID, fisID, or email

Useful fields to filter on organizations, individuals, or both.

Department ID ( the HCM id of a department ) - authors.organization.id.keyword
FISID ( the identifier of a person in the FIS database ) -  authors.fisID.keyword
Email ID ( The email ID that the individual specified via the FRPA interface ) - authors.email.keyword

##### This query demonstrates querying IBS by department id and then History by department name.:

    {"query":{"bool":{"must":[{"term":{"authors.organization.id.keyword":"10071"}},{"term":{"authors.organization.name.keyword":"History"}},{"term":{"authors.name.keyword":"Gutmann,+Myron++P."}},{"match_all":{}}]}},"sort":[{"publicationYear.keyword":{"order":"desc"}}],"from":0,"size":20,"aggs":{"mostSpecificType.keyword":{"terms":{"field":"mostSpecificType.keyword","size":115,"order":{"_count":"desc"}}},"authors.name.keyword":{"terms":{"field":"authors.name.keyword","size":120,"order":{"_count":"desc"}}},"publishedIn.name.keyword":{"terms":{"field":"publishedIn.name.keyword","size":115,"order":{"_count":"desc"}}},"publicationYear.keyword":{"terms":{"field":"publicationYear.keyword","size":125,"order":{"_count":"desc"}}},"authors.organization.name.keyword":{"terms":{"field":"authors.organization.name.keyword","size":115,"order":{"_count":"desc"}}},"authors.researchArea.name.keyword":{"terms":{"field":"authors.researchArea.name.keyword","size":115,"order":{"_count":"desc"}}},"cuscholarexists.keyword":{"terms":{"field":"cuscholarexists.keyword","size":115,"order":{"_count":"desc"}}},"amscore":{"range":{"field":"amscore","ranges":[{"to":0.001},{"to":100,"from":0.001},{"to":500,"from":100},{"from":500}]}}}}

##### This queries an department by ID AND a person by fisId:

    { "query": { "bool": { "must": [ { "term": { "authors.organization.id.keyword": "10071" } }, { "term": { "authors.fisId.keyword": "154905" } }, { "term": { "authors.name.keyword": "Gutmann,+Myron++P." } }, { "match_all": {} } ] } }, "sort": [ { "publicationYear.keyword": { "order": "desc" } } ], "from": 0, "size": 20, "aggs": { "mostSpecificType.keyword": { "terms": { "field": "mostSpecificType.keyword", "size": 115, "order": { "_count": "desc" } } }, "authors.name.keyword": { "terms": { "field": "authors.name.keyword", "size": 120, "order": { "_count": "desc" } } }, "publishedIn.name.keyword": { "terms": { "field": "publishedIn.name.keyword", "size": 115, "order": { "_count": "desc" } } }, "publicationYear.keyword": { "terms": { "field": "publicationYear.keyword", "size": 125, "order": { "_count": "desc" } } }, "authors.organization.name.keyword": { "terms": { "field": "authors.organization.name.keyword", "size": 115, "order": { "_count": "desc" } } }, "authors.researchArea.name.keyword": { "terms": { "field": "authors.researchArea.name.keyword", "size": 115, "order": { "_count": "desc" } } }, "cuscholarexists.keyword": { "terms": { "field": "cuscholarexists.keyword", "size": 115, "order": { "_count": "desc" } } }, "amscore": { "range": { "field": "amscore", "ranges": [ { "to": 0.001 }, { "to": 100, "from": 0.001 }, { "to": 500, "from": 100 }, { "from": 500 } ] } } } }

##### Finally the same query that uses the department ID for IBS and the email address for a person instead of the fisId

    { "query": { "bool": { "must": [ { "term": { "authors.organization.id.keyword": "10071" } }, { "term": { "authors.organization.name.keyword": "History" } }, { "term": { "authors.email.keyword": "Myron.Gutmann@Colorado.EDU" } }, { "match_all": {} } ] } }, "sort": [ { "publicationYear.keyword": { "order": "desc" } } ], "from": 0, "size": 20, "aggs": { "mostSpecificType.keyword": { "terms": { "field": "mostSpecificType.keyword", "size": 115, "order": { "_count": "desc" } } }, "authors.name.keyword": { "terms": { "field": "authors.name.keyword", "size": 120, "order": { "_count": "desc" } } }, "publishedIn.name.keyword": { "terms": { "field": "publishedIn.name.keyword", "size": 115, "order": { "_count": "desc" } } }, "publicationYear.keyword": { "terms": { "field": "publicationYear.keyword", "size": 125, "order": { "_count": "desc" } } }, "authors.organization.name.keyword": { "terms": { "field": "authors.organization.name.keyword", "size": 115, "order": { "_count": "desc" } } }, "authors.researchArea.name.keyword": { "terms": { "field": "authors.researchArea.name.keyword", "size": 115, "order": { "_count": "desc" } } }, "cuscholarexists.keyword": { "terms": { "field": "cuscholarexists.keyword", "size": 115, "order": { "_count": "desc" } } }, "amscore": { "range": { "field": "amscore", "ranges": [ { "to": 0.001 }, { "to": 100, "from": 0.001 }, { "to": 500, "from": 100 }, { "from": 500 } ] } } } }

Copyright and License
=====================

Copyright 2014 Cottage Labs.  Licensed under the MIT Licence
University of Colorado, Boulder - License: CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/)

twitter bootstrap: http://twitter.github.com/bootstrap/
MIT License: http://www.opensource.org/licenses/mit-license.php

