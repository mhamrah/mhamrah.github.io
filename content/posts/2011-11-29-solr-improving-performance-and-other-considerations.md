---
title: 'Solr: Improving Performance and Other Considerations'
author: Michael
type: post
date: 2011-11-29
url: /2011/11/solr-improving-performance-and-other-considerations/
dsq_thread_id:
  - 527367283
categories:
  - Programming
tags:
  - architecture
  - performance
  - search
  - solr
---
We use [Solr][1] as our search engine for one of our internal systems. It has been awesome; before, we had to deal with very messy sql statements to support many search criteria. Solr allows us to stick our denormalized data into an index and search on an arbitrary number of fields via an elegant, RESTful interface. It&#8217;s extremely fast, easy to use, and easy to scale. I wanted to share some lessons learned from our experience with Solr.

## Know Your Use Cases

There are two worlds of Solr: writing data (committing) and reading data (querying). Solr should not be treated like a database or some nosql solution; it is a search indexer built on top of Lucene. Treat it like a search indexer and not a permanent data store; it doesn&#8217;t behave like a database. There are plenty of tools to keep data in database in sync with Solr; the worst case scenario is you have to sync it yourself. You should know how heavy you will query it, how much you&#8217;ll write to it, and have a rough idea what your schema will be (but it doesn&#8217;t have to be 100%). Knowing your use cases will allow you to configure your instance and define your schema appropriately.

Solr offers a variety of ways to index and parse data; when you&#8217;re starting out, you don&#8217;t need to pick one. Solr has a great copyField feature that allows you to index the same data in multiple ways. This can be great for trying out new things or doing A/B comparisons. Once your patterns are well defined, you can tune your index and configuration as needed.

Our use cases are pretty straight forward; we simply need to search many different fields and aggregate results. We don&#8217;t need to deal with lexical analyzation or sorting on score. Our biggest issue was actually commits because we didn&#8217;t thoroughly vet our update patterns. Remember, Solr is about commits as much as it is about querying. You need to realize there will be some lag between when you update Solr and when you see the results. There are a large number of factors that go into how long that delay will be (it could be very quick), but it will be there, and you should design your system in knowing there will be a delay rather than trying to avoid it. The commit section covers why you shouldn&#8217;t try and commit on every update, even under moderate load.

## Know Configuration Options

Go through the solrconfig.xml and schema.xml files. It&#8217;s well documented and there are lots of good bits in there (solrconfig.xml is often missed!). The caches are what matter most, and explained in later sections. If you know your usage patterns you can get a good sense of how you can tune your caches for optimal results. Autowarming is also important; it allows Solr to reuse caches from previous indexes when things change.

Don&#8217;t forget that Solr sits on top of Java, so you should also tune the JVM as appropriate. This probably will revolve around how much memory to allocate to the JVM. Be sure to give as much as possible, especially in production.

## Understand Commits

You should control the number of commits being made to Solr. Load testing is important; you need to know how often and what happens when Solr will rebuilds an index. You shouldn&#8217;t commit on every update; you will surely hit memory and performance issues. When a commit occurs, an index and search warmer need to be built. A search warmer is a view onto an index. Caches may need to be pre-populated. Locking occurs. You don&#8217;t want to have that overhead if you don&#8217;t need it. If you have any post commit listeners those will also run. Finally, updating without forcing a commit is a lot faster than forcing a commit on update. The downside is simply that data will not be immediately available.

This is where autocommit comes into play. We use an autocommit every 5 seconds or 5000 docs. We never hit 5000 commits in less than 5 seconds; we just don&#8217;t want data to be too stale. 5000 docs allows us to re-index in production if we need to without killing the system. This ratio provides a good enough index time for searches to work appropriately without causing too many commits from choking the system. Again, know your usage patterns and you can get this number right.

## Search Warmers and Cold Searches

Solr caching works by creating a view on an index called a searcher. A commit will create a warming search to prep the index and the cache. How long this takes is tricky to say, but the more rows, indexable fields, and the more parsing that is done the longer it takes. The default is to only allow two warming searches at once, and depending on how you’re doing commits, you can easily surpass that limit. If you read the solrconfig.xml file you&#8217;ll see that 1-2 is useful for read-only slaves. So you&#8217;re going to want to increase this number on your main instance; but be aware, you can kill your available memory if you&#8217;re committing so much you have a high number of warmers.

By default Solr will block if a search warmer isn&#8217;t available. Depending on how and when you&#8217;re committing, you may not want this. For instance, if the first search is warming an index, it could be a while before it returns. Be sure to reuse old warmers and see if you can live with a semi-built index. This is all handled in the solrconfig.xml file. Read it!

## Increase Cache Sizes

Don&#8217;t forget out-of-the-box mode is not production mode. We&#8217;ve touched on a committing and search warmers. Cache sizes are another important aspect and should be as big as possible. This allows more warmers to be reused and offers a greater opportunity to search against cached search results (fq parameters) versus new query results (q parameters). The more we can cache the better; it also allows Solr to carry over search warmers when rebuilding indices which is very helpful.

## Lock Types

Luckily the default lock type is now &#8220;Native&#8221; which means Solr uses OS level locking. Previously it was single and this killed the system in concurrent update scenarios. Go native.

## Understand and Leverage Q and FQ parameters

Q is the original query, fq is a filter query.  For larger sets this is important. If the original query is cached an fq query will just search the cached original query, rather than the entire index.  So if you have an index with one million records, and a query returns 100k results, a q/fq combination will only search the 100k cached records. This is a big performance win. Ensure your cache settings are big enough for your usage patterns to create more cache hits.

## Minimize Use of Facets

Calculating facets is time consuming and can easily increase a search 2-5x than normal.  This is the slowest bottleneck we have with Solr (but still, it’s minimal compared to sql).  If you can avoid facets than do so.  If you can’t, only calculate them once on initial load, and design a UI that doesn’t need to refresh them (i.e. paging via ajax, etc).  When searching from a facet, use the fq parameter to minimize the set you&#8217;re searching on from your q query. This also reduces the required number of entries that are calculated for a facet and greatly increases performance.

## Avoid Dynamic Fields in Solr

This is more of an application architectural decision rather than anything else, and probably somewhat controversial. I feel you should avoid the use of dynamic fields and focus on defining your schema. I feel you can easily lose control over your schema if your model changes often as you have no base to work from. That can have unintended consequences depending on how you wrap your Solr instance and how you serialize and deserialize Solr data. It’s not too much up front work to define your schema during development that would call for the use of dynamic fields in production, unless of course your app necessitates using dynamic fields for one reason or another.

The other, more valid argument is that on a per-field level you can specify multi-values, required, and indexable fields. Solr handles multi valued and indexable fields differently on commits. If you are using dynamic fields and are indexing each one, and are not actually searching nor returning these fields, you have a really high and unnecessary commit cost. At the very least, consider turning off indexing for dynamic fields if you don&#8217;t need it.

## Use Field Lists

You should always specify what data you want returned from the query with fl (field list). This is extremely important!  Depending on how you’ve set up your schema, you probably have a ton of fields you don’t actually need returned to the UI.  This is common when you are indexing the same field with different parsers via the copyField functionality. Use fl to get back only the data you need- this will greatly reduce the amount data (and network traffic) returned, and speed up the query because Solr will not have to fetch unnecessary fields from its internal storage. In a high-read environment, you can greatly reduce both memory and network load by trimming the fat from your dataset.

## Have a Reindex strategy

There will come a time when you need to reindex your Solr instance. Most likely this will be when you&#8217;re releasing a new feature. It&#8217;s important to have a reindexing strategy ready to go. Let&#8217;s say you add a new field to your UI which you want to search on. You release your code, but that field is not in Solr yet so you get no results. Or, you get a doc back from Solr, you deserialize it to your object model, and get an error because you expect the field to be there and it&#8217;s not. You must prepare for that. You could change your schema file, reindex in a background process, and then release code when ready. In this scenario make sure you can reindex without killing the system. It&#8217;s also important to know how long it will take. Having to reindex like this may not be practical if takes a couple of days. You could also reindex to a second, unused Solr instance, and when you deploy you cut over to the new instance. By looking at your db update timestamps you can sync any missed data. (Remember how I said Solr is not a data store? This is a reason.)

## Final Thoughts

Remember that data in Solr needs to be stored, indexed and returned. If you are only using dynamic fields, indexing all of them, defining copyField settings left and right and returning all that data because you are not using field lists (and potentially calculating facets on everything), you are generating a lot of unnecessary overhead. Keep it small and keep it slim. You&#8217;ll lower your storage needs, your memory requirements, and your result set. You&#8217;ll speed up commits as well.

 [1]: http://lucene.apache.org/solr/
