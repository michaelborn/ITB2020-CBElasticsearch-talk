# ITB 2020: Power Up Your Search with CBelasticsearch

> by [Michael Born](#about-me) - Software Engineer @ Ortus Solutions Corp.

If you think database full-text search is slow and bare-bones, you're right. ElasticSearch is becoming the de-facto standard for lightning-fast search, and with cbElasticSearch, the Ortus-built ElasticSearch API client, you can easily integrate with ElasticSearch in your ColdBox application to provide your users with fast and powerful search. This session will cover ElasticSearch usage from installation and configuration to indexing documents, searching using cbElasticSearch's intuitive API and returning "Did you mean?" suggestions for a fantastic user experience.

## Table Of Contents
- [ITB 2020: Power Up Your Search with CBelasticsearch](#itb-2020-power-up-your-search-with-cbelasticsearch)
  - [Table Of Contents](#table-of-contents)
  - [About Me](#about-me)
  - [The State of Search](#the-state-of-search)
    - [Expectations](#expectations)
    - [Current Solutions](#current-solutions)
      - [cfSearch](#cfsearch)
      - [Database Search](#database-search)
  - [Elasticsearch](#elasticsearch)
  - [Installation and Configuration](#installation-and-configuration)
    - [Installing Elasticsearch](#installing-elasticsearch)
    - [Installing CBelasticsearch](#installing-cbelasticsearch)
    - [Configuring Elasticsearch](#configuring-elasticsearch)
  - [Managing your Elasticsearch Index](#managing-your-elasticsearch-index)
    - [Creating an Index](#creating-an-index)
    - [Updating an Index](#updating-an-index)
    - [Reindexing](#reindexing)
    - [Index Migrations](#index-migrations)
    - [Deleting an Index](#deleting-an-index)
  - [Working with Documents](#working-with-documents)
    - [Saving a Document](#saving-a-document)
    - [Updating a Document](#updating-a-document)
    - [Deleting a Document](#deleting-a-document)
  - [Searching with Elasticsearch](#searching-with-elasticsearch)
    - [Requires proper mapping or may not find correct results](#requires-proper-mapping-or-may-not-find-correct-results)
    - [Term vs Match](#term-vs-match)
      - [Term](#term)
      - [Match](#match)
      - [Fuzzy searching](#fuzzy-searching)
    - [Filter vs Query](#filter-vs-query)
      - [Filter](#filter)
      - [Query](#query)
    - [Boolean searching](#boolean-searching)
      - [`must`](#must)
      - [`should`](#should)
  - [Searching Auto-generated text fields](#searching-auto-generated-text-fields)

## About Me

* Started web development at a web design agency in 2011
* ♡ Married two years
* 🔨 In the middle of remodeling our first house!
* 🐦 Twitter - [@michaelborn_me](https://twitter.com/michaelborn_me)
* ✎ I blog at [MichaelBorn.me](https://michaelborn.me)

## The State of Search

The year is 2020. Search is a first-class citizen - e.g. the majority of websites are expected to have a functional search.

### Expectations

* More search
* Faster search
* Accurate results
* Typo correction
* Auto complete

### Current Solutions

> "You can't solve a 2020 problem with a 2010 solution" - Michael Born. Used by permission.

#### cfSearch

* Built into the CF engine and hard to update/upgrade/configure.

#### Database Search

* DB's fit a totally different use case.

## Elasticsearch

* Fast
* Intelligent
* Built to scale

## Installation and Configuration

### Installing Elasticsearch

Note: Seems to have issues if you don't specify a version tag. E.g. `:latest` pulls don't work for me. 🤷

```js
docker pull elasticsearch:7.6.2
docker run -d -p 9200:9200 --name elastic_lion  -e "discovery.type=single-node" elasticsearch:7.6.2
```

### Installing CBelasticsearch

```js
box install CBelasticsearch
```

### Configuring Elasticsearch

Two ways to do this - the first is using environment variables. You can use [`commandbox-dotenv`](https://forgebox.io/view/commandbox-dotenv) to read your `.env` file into your web server. Once those env vars are loaded in place, CBelasticsearch will automatically use those to configure the Elasticsearch connection.

```js
// .env

# Elasticsearch connection
ELASTICSEARCH_PROTOCOL=http
ELASTICSEARCH_HOST=127.0.0.1
ELASTICSEARCH_PORT=9200

# Connection authentication, if configured
ELASTICSEARCH_USERNAME=myUser
ELASTICSEARCH_PASSWORD=CHANGE_ME

# Optional settings
# Default index name
ELASTICSEARCH_INDEX=sticksandstones
# The default number of shards to use when creating an index
ELASTICSEARCH_SHARDS
# The default number of index replicas to create
ELASTICSEARCH_REPLICAS
# The maxium number of connections, in total for all Elasticsearch requests
ELASTICSEARCH_MAX_CONNECTIONS
# Read timeout - the read timeout in milliseconds
ELASTICSEARCH_READ_TIMEOUT
# Connection timeout - timeout attempts to connect to elasticsearch after this timeout
ELASTICSEARCH_CONNECT_TIMEOUT
```

You can also configure this in CFML using your Coldbox config file located at `config/Coldbox.cfc`:

```js
// config/Coldbox.cfc
moduleSettings = {
    CBelasticsearch = {
        hosts = [
            {
                serverProtocol: "http",
                serverName: "127.0.0.1",
                serverPort: "9200"
            }
        ],
        defaultCredentials = {
            "username" : "esUser",
            "password" : "CHANGE_ME"
        },
        // The default index
        defaultIndex   : "sticksandstones";
    }
};
```

## Managing your Elasticsearch Index

### Creating an Index

Using the Elasticsearch mapping syntax:

```js
var indexBuilder = getInstance( "IndexBuilder@CBelasticsearch" ).new(
    "review",
    {
        "properties" : {
            "title" : { "type" : "text" },
            "rating" : { "type" : "integer" },
            "body" : { "type" : "text" }
        }
    }
).save();
```

Using the MappingBuilder

```js
var myNewIndex = getInstance( "IndexBuilder@CBelasticsearch" ).new(
    "review",
    function( builder ){
        return {
            "_doc" = builder.create( function( mapping ) {
                mapping.text( "title" );
                mapping.integer( "rating" );
                mapping.text( "body" );
            });
        }
    }
).save();
```

### Updating an Index

Use the IndexBuilder `.update()` method to issue what we call a "PUT mapping" - so named because the API call issued to update the index mapping uses the `PUT` HTTP method.

```js
var updatedIndex = getInstance( "IndexBuilder@CBelasticsearch" ).update(
    "review",
    function( builder ){
        return {
            builder.create( function( mapping ) ){
                mapping.text( "title" );
                mapping.integer( "rating" );
                mapping.text( "body" );
                mapping.
            };
        }
    }
)
```

### Reindexing

If you need to make any serious updates to an Elasticsearch index, you'll need to reindex it. Basically, ES isn't great for mass updates of documents and/or changing document field types.

For this, you'll need to create a new index and reindex (e.g. "pour") the documents from the old index into the new.

```js
getInstance( "Client@CBelasticsearch" ).reindex( "books", "books" );
```

### Index Migrations

1. create a new index with the new config. The config lives in the migration to keep track of changes.
2. Reindex data from the old index to the new index.
3. Simultaneously swap and alias from the old index to the new index.

### Deleting an Index

If you need to delete an index, here's how you can do that with the `Client` object:

```js
getInstance( "IndexBuilder@CBelasticsearch" )
    .setIndexName( "books" )
    .delete();
```

## Working with Documents

Use the `Document` object to create and manage documents.

### Saving a Document

There are a few ways to generate a document "memento" and pass it to the `Document` object, but my favorite by far is the `populate` method. Nice and sweet - call `.new()`, `.populate()`, and `.save()` and you have your new ES document!

```js
var newDocument = getInstance( "Document@CBelasticsearch" )
                    .new( "receipts" )
                    .populate({
                        "_id" : "E1",
                        "name" : "Lowes E1",
                        "subtotal" : 590.77,
                        "tax" : 35.21,
                        "total" : 625.98
                    })
                    .save();
```

### Updating a Document

Let's say the user just filled out the optional `description` field on their receipt form. To update the document in Elasticsearch, we need to retrieve the document, verify that it exists, set the new value, and save the document back into ES.

```js
var document = getInstance( "Document@CBelasticsearch" )
                .get( "E1", "receipts" );
if ( !isNull( document ) ){
    document.setValue( "description", "Lowes receipt for plumbing supplies" )
            .save();
} else {
    // Holler via LogBox
    logbog.getRootLogger().warn( "Document E1 not found" );
}
```

### Deleting a Document

What happens when a user inputs an item that they then decide to delete? How do you clean up that record from Elasticsearch?

```js

var document = getInstance( "Document@CBelasticsearch" )
                .get( "E1", "receipts" );
if ( !isNull( document ) ){
    document.delete();
} else {
    // Holler via LogBox
    logbog.getRootLogger().warn( "Document E1 not found" );
}
```

## Searching with Elasticsearch

Here's where the fun begins!

All searching in CBelasticsearch is done via the `SearchBuilder` component. To issue a search, create a new instance of `SearchBuilder` and call `.new()` to set the name of the index to search.

```js
var searchResults = getInstance( "SearchBuilder@CBelasticsearch" )
                .new( "receipts" )
                .match( "name", "Lowes" )
                .execute();
```

### Requires proper mapping or may not find correct results

Before we get too much farther, it is important that you understand the following in order to receive accurate search results:

1. [how your index is mapped](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) (e.g. how the index fields and field types are defined) and 
2. [how the Elasticsearch analyzer works](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)

I'll do my best to explain these topics as we come to them, but the best explanations can be found in [the Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html).

### Term vs Match

#### Term

Use `term` queries for `keyword` fields. Do **not** use on `text` fields or you will not get any results, buecause `text` fields are analyzed during indexing and not stored in their original form.

Here's an example of a `term` query using the Elasticsearch query JSON syntax:

```js
{
    "query" : {
        "term" : { "username" : "michael.born" }
    }
}
```

And using CBelasticsearch, we would simply call `.term()` on the `SearchBuilder` component - making sure to pass our field name and a value to search for.

```js
var searchResults = getInstance( "SearchBuilder@CBelasticsearch" )
                .new( "books" )
                .term( "username", "michael.born" )
                .execute();
```

#### Match

Use `match` for analyzed search queries. The elasticsearch analyzer will perform manipulations on your search phrase/word to break it down for improved searching. The exact results depend on which analyzer is set on the field you are searching, but Elasticsearch may remove "stop words" such as "a" and "the", break compound words like "running" down into their stems like "run", and lowercase all content.

For this reason, it's important to use `match` queries instead of `term` queries on all `text` fields, as `text` fields are analyzed and thus not stored in their original format.

Here's an example of a `match` query in pure Elasticsearch syntax:

```js
{
    "query" : {
        "match" : { "review" : "Star Wars" }
    }
}
```

And here's the CBelasticsearch syntax:

```js
var searchResults = getInstance( "SearchBuilder@CBelasticsearch" )
                .new( "books" )
                .match( "review", "Star Wars" )
                .execute();
```

#### Fuzzy searching

Fuzzy is any non-exact matching.

You know, "fuzzy". Unclear. Not a hard black/white border.

Term queries are NOT fuzzy - they are black and white. It either is `id=1234` or not.

Match queries allow fuzzy search strings - they can match part of a phrase. Like `name=Michael*` will catch both `Michael Born` and `Michael Jordan`.

We call this "Fuzziness".

https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness

### Filter vs Query

The key word here is relevance. The Query context evaluates a "score" for each matching document, while the Filter context does not - it only cares about inclusion or exclusion.

#### Filter

By default Elasticsearch determines a relevance score for all documents matched in the search query. This relevance score determines the ranking order of documents returned from the search. To alter the query results without affecting relevance (say, to exclude all documents marked `isActive: false`), we need to use the `filter` construct to denote that the nested queries should be performed without affecting the relevance ranking.

```js
{
    "query" : {
         "filter" : {
             "term" : { "isActive" : true }
         }
    }
}
```

We can accomplish this in CBelasticsearch using `SearchBuilder`'s `filterMatch()` method.

```js
var searchResults = getInstance( "SearchBuilder@CBelasticsearch" )
                .new( "books" )
                .filterMatch( "isActive", true )
                .execute();
```

> Note: This `filterMatch()` function is not exactly equivalent to the above elasticsearch JSON syntax. Under the hood, `searchBuilder.filterMatch` uses a boolean `must[]` array, but the end result should be the same.

#### Query

```js
{
    "query" : {
       "term" : { "year" : 1977 }
    }
}
```

### Boolean searching

When you have more than one search criteria to look for, you have to make the decision: should this additional criteria *narrow* my search, or *expand* it? Elasticsearch lets you choose between the two using boolean search operators: `must` and `should`. (AND and OR, respectively.)

#### `must`

A `must` clause in Elasticsearch, like an `AND` SQL boolean, specifies that *all* of the criteria *must* match.

```js
{
    "query" : {
        "bool" : {
            "must" : [
                { "match" : { "description" : "Lowes" } }
                { "range" : { "total" : { "gt" : "500" } } }
            ]
        }
    }
}
```

#### `should`

A `should` clause in Elasticsearch, like an SQL `OR` boolean, specifies that at least *one* of the criteria *should* match.

```js
{
    "query" : {
        "bool" : {
            "should" : [
                { "match" : { "description" : "Lowes" } }
                { "range" : { "total" : { "gt" : "500" } } }
            ]
        }
    }
}
```

## Searching Auto-generated text fields

Any field less than 156 (maybe 255?) characters which is auto-mapped (e.g. not explicitly set) to `text` will have a subfield of `keyword` you can reference for exact `term` searches. This helps mitigate some of the effects of auto-mapped fields set to a `text` field type.

```js
{
    "query" : {
    	"bool" : {
        	"filter" : {
            	"term" : { "author.firstName.keyword" : "Michael Born" }
            }
        }
    }
}
```