# ITB 2020: Power Up Your Search with CBelasticsearch

> by [Michael Born](#about-me) - Software Engineer @ Ortus Solutions Corp.

If you think database full-text search is slow and bare-bones, you're right. ElasticSearch is becoming the de-facto standard for lightning-fast search, and with cbElasticSearch, the Ortus-built ElasticSearch API client, you can easily integrate with ElasticSearch in your ColdBox application to provide your users with fast and powerful search. This session will cover ElasticSearch usage from installation and configuration to indexing documents, searching using cbElasticSearch's intuitive API and returning "Did you mean?" suggestions for a fantastic user experience.

## About Me

* Started web development at a web design agency in 2011
* 💑 Married in 2018
* 🚀 Joined Ortus August 2019
* 🏠 Just bought our first house!
* 🔨 And I'm ripping it apart / remodeling it myself! 🤣
* 🐦 Twitter - [@michaelborn_me](https://twitter.com/michaelborn_me)
* 📝 I blog at [MichaelBorn.me](https://michaelborn.me)

## Table Of Contents
- [ITB 2020: Power Up Your Search with CBelasticsearch](#itb-2020-power-up-your-search-with-cbelasticsearch)
  - [About Me](#about-me)
  - [Table Of Contents](#table-of-contents)
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
  - [Elasticsearch Indexing Concepts](#elasticsearch-indexing-concepts)
    - [What Is An Index?](#what-is-an-index)
    - [Indexing a Document](#indexing-a-document)
  - [Managing your Elasticsearch Index](#managing-your-elasticsearch-index)
    - [Creating](#creating)
    - [Updating](#updating)
    - [Migrating](#migrating)
    - [Reindexing](#reindexing)
    - [Deleting](#deleting)
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
  - [Random Tips](#random-tips)
  - [Searching Auto-generated text fields](#searching-auto-generated-text-fields)
  - [Resources](#resources)

## The State of Search

The year is 2020. Search is a first-class citizen - e.g. the majority of websites are [*expected*](#expectations) to have a functional search.

### Expectations

* More search
* Faster search
* Accurate results
* Typo correction
* Auto complete

### Current Solutions

> "You can't solve a 2020 problem with a 2010 solution" - Michael Born. Used by permission.

#### cfSearch

CFSearch is a 2010 solution to a 2020 problem.

* Built into the CF engine
  * Difficult or impossible to update/upgrade
  * Difficult or impossible to configure
  * Very difficult to access the underlying search engine
* Limited query language
  * One can only cram so much syntax into a single tag
* Difficult to configure or debug a remote machine
  * Advanced options are set in `.txt` and `.xml` files
  * Requires shell or GUI access to the machine
* No built-in scaling mechanisms

#### Database Search

Database search is a 2001 solution to a 2020 problem.

* DB's fit a completely different use case.
  * Designed for fast writes as well as reads
  * Designed around ACID principles
  * Specifically, atomicity - queries usually need joins between multiple tables
  * Usually stored on disk
* Querying full text is painfully slow.
* No natural language search
  * Can't match Home to House
* No ordering by search relevance

> **Note:** A relational database is *still* the best way to store and manage large amounts of data in an accurate, consistent and easily editable manner. I am not hating on databases - they are awesome, just not designed nor fit for powerful search applications.

## Elasticsearch

* Fast
  * No joins necessary
  * Uses RAM for quicker data reads
* Intelligent
  * Analyzes text for natural language search
  * Can match similar word forms like "running" and "runner"
  * Can ignore words like "the", "and", or "it"
* Built to scale
* Not limited by DB constraints such as "atomicity"
  * We don't care if some data is redundant!
  * (i.e. not "atomic")
  * Why? Because redundant data is *fast*.

## Installation and Configuration

Once you've made the decision to move forward with Elasticsearch it is incredibly easy to get started. Thanks to [Docker](https://hub.docker.com/_/elasticsearch), [CommandBox](https://commandbox.ortusbooks.com/), and [CBElasticsearch](https://forgebox.io/view/cbelasticsearch) itself, we can have a ColdBox app connecting to a new Elasticsearch server in just a few commands.

### Installing Elasticsearch

Docker is awesome! Use `docker pull` to download the latest version - in this case, `7.6.2`.

> Note: The `elasticsearch` docker image does not like the industry-standard `:latest` tag, so you'll have to look up the latest version and specify it manually in the download string. 🤷

```bash
docker pull elasticsearch:7.6.2
docker run -d -p 9200:9200 -e "discovery.type=single-node" elasticsearch:7.6.2
```

The important piece here is the port mapping, which uses your machine port (host port) `9200` to map to the container port `9200` which is the default for Elasticsearch connections.

Also note we are passing an environment variable to Elasticsearch which instructs Elasticsearch to only build and configure a single node. This is simply because you shouldn't need more than a single node for demo purposes!

### Installing CBelasticsearch

Thank CommandBox (and [Brad Wood](https://forgebox.io/view/cbelasticsearch)) for the ability to download open source CFML packages (such as [CBElasticsearch](https://forgebox.io/view/cbelasticsearch)) in a single command.

```bash
box install cbelasticsearch
```

### Configuring Elasticsearch

Two ways to do this - the first is using environment variables. You can use [`commandbox-dotenv`](https://forgebox.io/view/commandbox-dotenv) to read your `.env` file into your web server. Once those env vars are loaded in place, CBelasticsearch will automatically use those to configure the Elasticsearch connection.

```bash
# .env
# Elasticsearch connection
ELASTICSEARCH_PROTOCOL=http
ELASTICSEARCH_HOST=127.0.0.1
ELASTICSEARCH_PORT=9200

# Connection authentication, if configured
ELASTICSEARCH_USERNAME=myUser
ELASTICSEARCH_PASSWORD=CHANGE_ME
```

There are more optional settings if you need to fine-tune your Elasticsearch config (and don't want to use `moduleSettings`  in `config/ColdBox.cfc`):

```bash
# .env
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

## Elasticsearch Indexing Concepts

Elasticsearch works the way it does because of the powerful data methodologies underlying all that Java. Documents are not stored as-is, like a file, nor are they merely separated into fields as with a database. Elasticsearch uses an index to organize all related documents and track searchable keywords and phrases across the entire set of documents.

### What Is An Index?

> I highly recommend reading the [Elasticsearch documentation on documents and indices](https://www.elastic.co/guide/en/elasticsearch/reference/current/documents-indices.html).

An index can refer to two things: one very generic, and the other **very** specific.

First, the term "index" can mean a collection of documents organized around a specific type, such as "Autos", "Logs", "Books" or "Reviews". You can think of this (in a high-level sense) as a database table - e.g. a set of rows with a defined structure and purpose. When referring to an Elasticsearch "index" in this document/repo/talk, I'm usually referring to this type of index.

Secondly, "index" is both the process (in verb form) and the result (index as a noun) of analyzing the text values within documents in order to build an mapping (ok, an index) of searchable keywords (called "tokens") of each document. When a search query is executed, Elasticsearch will analyze your search terms in a similar fashion and look them up within the index to rapidly determine which documents have those same tokens.

For example, the phrase `the quick brown fox jumps over the lazy dog` can be broken down into the following rudimentary index:

```js
{
    "the" : { "frequency" : 2 },
    "quick" : { "frequency" : 1 },
    "brown" : { "frequency" : 1 },
    "fox" : { "frequency" : 1 },
    "jumps" : { "frequency" : 1 },
    "over" : { "frequency" : 1 },
    "lazy" : { "frequency" : 1 },
    "dog" : { "frequency" : 1 }
}
```

Note that since `the` is used twice, this sentence is twice as relevant in a search for `the` as it is for a search for `fox`. However, much of the time it is useful to [configure Elasticsearch to remove "stopwords"](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html) - the Elasticsearch term for generic, non-useful English words such as `"the"`, `"and"` and `"or"`.

Furthermore, Elasticsearch supports the ability to "stem" tokens during the indexing process. Stemming breaks complex words down into their root form, making it easy to match `"jumps"` with `"jumping"`.

With those two changes, let's look at our document tokens again:

```js
{
    "quick" : { "frequency" : 1 },
    "brown" : { "frequency" : 1 },
    "fox" : { "frequency" : 1 },
    "jump" : { "frequency" : 1 },
    "over" : { "frequency" : 1 },
    "lazy" : { "frequency" : 1 },
    "dog" : { "frequency" : 1 }
}
```

Since the `"the"` token is removed, the document is now more relevant to the word `"dog"`, for example, because the ratio of the word `"dog"` to the entire document just went up slightly.

Secondly, the token `"jumps"` has been broken down (e.g. "stemmed") into `"jump"`. While this is not a drastic change, it is indeed a large step forward. Searching the index by the following phrase: `"dog jumping"` will match the document because when the search phrase is analyzed it is *also* broken down into root words, and thus a search for `"dog jumping"` becomes a search for `"dog jump"`. Once that analysis is done, it is easy to find our document by those tokens.

### Indexing a Document

## Managing your Elasticsearch Index

### Creating

We can create a new index using the Elasticsearch mapping syntax:

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

But we can clean this up a little by using the `MappingBuilder` object:

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

### Updating

Use the `IndexBuilder`'s `.update()` method to issue what we call a "PUT mapping" - so named because the API call issued to update the index mapping uses the `PUT` HTTP method.

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

### Migrating

When the time comes to update an Elasticsearch index due to new fields, changes to field types, or removal of old fields, you will need to perform a "migration" to convert data accurately from the old format to the new. Here's the general flow [Eric Peterson](https://twitter.com/_elpete) uses and recommends when migrating an index to a new mapping configuration:

1. Create a new index with the new mapping configuration. The mapping config should be created and saved in a migration file (perhaps in `resources/migrations/elasticsearch`, for example`) to keep track of changes.
2. [Reindex data from the old index to the new index](#reindexing).
3. Simultaneously swap any alias(es) from the old index to the new index.

### Reindexing

If you need to make any serious updates to an Elasticsearch index, you'll need to "reindex" it. Basically, Elasticsearch isn't great for mass updates of documents and/or changing document field types. (Remember that Elasticsearch is not a database? Here's where Elasticsearch requires a bit more work than a relational DB. The pros still outweigh the cons.)

```js
getInstance( "Client@CBelasticsearch" ).reindex( "books", "books" );
```

To reindex means to create a new index and basically pour the documents from the old index into the new.

### Deleting

If you need to delete an index, here's how you can do that with the `Client` object:

```js
getInstance( "IndexBuilder@CBelasticsearch" )
    .setIndexName( "books" )
    .delete();
```

It's not hard. 😉 You can also use the `deleteIndex()` method in the `Client` object if you so desire.

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

I'll do my best to explain these topics as we come to them, (actually, see the [indexing explanation](#what-is-an-index)) but the best explanations can be found in [the Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html).

### Term vs Match

#### Term

Use `term` queries for `keyword` fields. Do **not** use on `text` fields or you will not get any results, because `text` fields are analyzed during indexing and not stored in their original form.

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

Match queries allow fuzzy search strings - they can match part of a phrase. Like `name=M*l` will catch both `Michael` and `Marshall`.

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

## Random Tips

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

## Resources

For more learning, check out any or all the following:

* [CBElasticsearch documentation](https://cbelasticsearch.ortusbooks.com/)
* [Easy Elasticsearch with CBElasticsearch](https://www.slideshare.net/ortussolutions/itb2019-easy-elasticsearch-with-cbelasticsearch-jon-clausen) - ITB2019 talk by Jon Clausen. This was my first intro to Elasticsearch.
* [Enterprise Search with ColdFusion Solr](https://dsirucek.files.wordpress.com/2012/05/enterprise-search-with-coldfusion-solr-final.pdf) - This is why we use Elasticsearch - we are not limited by a two-tag interface.