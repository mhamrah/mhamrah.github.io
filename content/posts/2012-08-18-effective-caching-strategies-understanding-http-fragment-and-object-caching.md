---
title: 'Effective Caching Strategies: Understanding HTTP, Fragment and Object Caching'
author: Michael
type: post
date: 2012-08-18
url: /2012/08/effective-caching-strategies-understanding-http-fragment-and-object-caching/
dsq_thread_id:
  - 810252868
categories:
  - Programming
tags:
  - architecture
  - cache
  - caching
  - performance
  - scaling
---
Caching is one of the most effective techniques to speed up a website and has become a staple of modern web architecture. Effective caching strategies will allow you to get the most out of your website, ease pressure on your database and offer a better experience for users. Yet as the old [adage says][1] caching&#8211;especially invalidation&#8211;is tricky. How to deal with dynamic pages, deciding what to cache, per-user personalization and invalidation are some of the challenges which come along with caching.

### Caching Levels

There a three broad levels of caching:

  * _[HTTP Caching][2]_ allows for full-page caching via HTTP headers on URIs. This must be enabled on all static content and should be added to dynamic content when possible. It is the best form of caching, especially for dynamic pages, as you are serving generated html content and your application can effectively leverage reverse-proxies like [Squid][3] and [Varnish][4]. [Mark Nottingham&#8217;s great overview on HTTP Caching is worth a read][5].

  * _Fragment Caching_ allows you to cache page fragments or partial templates. When you cannot cache an entire http response, fragment caching is your next best bet. You can quickly assemble your pages from pre-generated html snippets. For a page involving disparate dynamic content you can build your result page from cached html fragments for each section. For listing pages, like search results, you can build the page from html fragments for each id and not regenerate markup. For detail pages you can separate less-volatile or common sections from high-volatile or per-user sections.

  * _Object Caching_ allows you to cache a full object (as in a model or viewmodel). When you must generate html for each user/request, or when your objects are shared across various views, object caching can be extremely helpful. It allows you to better deal with expensive queries and lessen hits to your database.

The goal is to make your response times as fast as possible while lessening load. The more html (or data) you can push closer to the end-user the better. HTTP caching is better than fragment caching: you are ready to return the rendered page. When combined with a CDN even dynamic pages can be pushed to edge locations for faster response times. Fragment caching is better than object caching: you already have the rendered html to build the page. Object caching is better than a database call: you already have the cached query result or denormalized object for your view. The deeper you get in the stack (the closer to the datastore) the more options you have to vary the output. Consequently the more expensive and longer the operation will take.

### Break Content Down; Cache for Views

A cache strategy is dependent on breaking content down to store and reuse later. The more granular you can get the more options you have to serve cached content. There are two main dimensions: what to cache and whom to cache for. It is difficult to HTTP cache a page with a &#8220;Hello, {{ username }}&#8221; in the header for all users. However if you break your users down into logged-in users and anonymous users you can easily HTTP cache your homepage for just anonymous users using the _vary_ http-header and defer to fragment caching for logged-in users.

Cache key naming strategies allow you to vary the _what_ with the _who for_ in a robust way by creating multiple versions of the same resource. A cache key could include the role of the user and the page, such as _role:page:fragement:id_, as in _anon:widget_detail:widget:1234_ and serve the _widget detail_ html fragment to anonymous users. The same widget could be represented in a search detail list via _anon:widget_search:widget:1234_. When widget 1234 updates both keys are invalidated. Most people opt for object caching for an easy win with dynamic pages, specifically by caching via a primary key or id. This can be helpful, but if you break down your content into the _what_ and _who for_ with a good key naming strategy you can leverage fragment caching and save on rendering time.

The _vary_ http header is very helpful for dealing with HTTP caching and is not used widely enough. By varying URIs based on certain headers (like authorization or a cookie value) you can cache different representations for the same resource in a similar way to creating multiple keys. Think of the cache key as the URI plus whatever is set in the _vary_ header. This opens up the power of HTTP caching for dynamic or per-user content.

You are ready to deliver content quickly when you think about your cache in terms of views and not data. Cache a denormalized object with child associations for easy rendering without extra lookups. Store rendered html fragments for sections of a page that are common to users on otherwise specific content. &#8220;Popular&#8221; and &#8220;Recent&#8221; may be expensive queries; storing rendered html saves on processing time and can be injected into the main page. You can even reuse fragments across pages. A good cache key naming strategy allows for different representations of the same data which can easily be invalidated.

### Cache Invalidation

Nobody likes stale data. As you think about caching think about what circumstances to invalidate the cache. Time-based expirations are convenient but can usually be avoided by invalidating caches on create and update commands. A good cache key naming strategy helps. Web frameworks usually have a notion of &#8220;callbacks&#8221; to perform secondary actions when a primary action takes place. A set of fragment and object caches for a widget could be invalidated when a record is updated. If cache values are granular enough you could invalidate sections of a page, like blog comments, when a comment is added and not expire the entire blog post.

HTTP Etags provide a great mechanism for dealing with stale HTTP requests. Etags allow a more invalidation options than the basic if-modified-since headers. When dealing with Etags the most important thing is to avoid processing the entire request simply to generate the Etag to validate against (this saves network bandwidth but does not save processing time). Caching Etag values against URIs are a good way to see if an Etag is still valid to send the proper 304 NOT MODIFIED response as quickly as possible in the request cycle. Depending on your needs you can also cache sets of Etag values against URIs to handle various representations.

If you must rely on time-based expiration try to add expiration callbacks to keep the cache fresh, especially for expensive queries in high-load scenarios.

### Edge Side Includes: Fragment Caching for HTTP

Edge Side Includes are a great way of pushing more dynamic content closer to users. ESIs essentially give you the benefits of fragment caching with the performance of HTTP caching. If you are considering using a tool like [Squid][3] or [Varnish][4] ESIs are essential and will allow you to add customized content to otherwise similar pages. The _user panel_ in the header of a page is a classic example of an ESI usage. If the user panel is the only variant of an otherwise common page for all users, the common elements could be pulled from the reverse-proxy within milliseconds and the &#8220;Welcome, {{USER}}&#8221; injected dynamically as a fragment from the application server before sending everything to the client. This bypasses the application stack lightening load and decreasing processing time.

### Distributed or Centralized Caches are Better

Distributed and/or centralized caches are better than in-memory application server cache stores. By using a distributed cache like [Memcache][6], or a centralized cache store like [Redis][7], you can drop duplicate data caches to make caching and invalidating objects easier. Even though caching objects in a web app&#8217;s memory space is convenient and reduces network i/o, it soon becomes impractical in a web farm. You do not want to build up caches per-server or steal memory space away from the web server. Nor do you want to have to hunt and gather objects across a farm to invalidate caches. If you do not want to support your own cache farm, there are plenty of SaaS services to deal with caching.

### Compress When Possible

Compressing content helps. Memory is a far more valuable resource for web apps than cpu cycles. When possible, compress your serialized cache content. This lowers the memory footprint so you can put more stuff in cache, and lightens the transfer load (and time) between your cache server and application server. For HTTP caching the helpful _vary_ http header can also be used to cache content for browsers supporting compression and those that don&#8217;t. For object caching, only store what you need in the cache. Even though compression helps reduce the footprint, not storing extraneous data further reduces the footprint and saves serialization time.

### NoSQL to the Rescue

One of the interesting trends I am reading about is how certain NoSQL stores are eliminating the need for separate cache farms. NoSQL solutions are beneficial for a variety of reasons even though they create significant data-modeling challenges. What NoSQL solutions lack in the flexibility of representing and accessing data (i.e. no joins, minimal search) they can make up in their distributed nature, fault-tolerance, end access efficiency. When you model your data for your views, putting the burden on storing data in the same way you want to get it out, you&#8217;re essentially replacing your denormalized memory-caching tier with a more durable solution. Cassandra and other Dynamo/Bigtable type stores are key-value stores, similar to cache stores, with the value part offering some sort of structured data type (in the case of Cassandra, sorted lists via column families). MongoDb and Redis, (not Dynamo inspired) offer similar advantages; Redis&#8217; sorted sets/sorted lists offer a variety of solutions for listing problems, MongoDb allows you to query objects.

If you are okay with storing (and updating) multiple-versions of your data (again, you are caching for views) you can cut the two-layer approach of separate cache and data stores. The trick is storing everything you need to render a view for a given key. Searches could be handled by a search-server like Solr or ElasticSearch; listing results could be handled by maintaining your own index via a sorted-list value via another key. When using Cassandra you&#8217;d get fast, masterless, and scalable persistant storage. In general this approach is only worthwhile if your views are well-defined. The worst thing you want to do is refactor your entire data model when your views change!

### How Web Frameworks Help

There is always debate on differences between frameworks and languages. One of the things I always look for is how easy it is to add caching to your application. Rails offers great support for caching, and the [Caching with Rails][8] guide is worth a read no matter what framework or language you use. It easily supports fragment caching in views via content blocks, behind-the-scene action caching support, has a pluggable cache framework to use different stores, and most importantly has an extremely flexible invalidation framework via model observers and cache sweepers. When choosing any type of framework, &#8220;how to cache&#8221; should be a bullet point at the top of the list.

 [1]: http://martinfowler.com/bliki/TwoHardThings.html
 [2]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html
 [3]: http://www.squid-cache.org/
 [4]: https://www.varnish-cache.org/
 [5]: http://www.mnot.net/cache_docs/
 [6]: http://memcached.org/
 [7]: http://redis.io
 [8]: http://guides.rubyonrails.org/caching_with_rails.html
