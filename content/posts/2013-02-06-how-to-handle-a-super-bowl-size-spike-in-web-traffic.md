---
title: How to Handle a Super Bowl Size Spike in Web Traffic
author: Michael
type: post
date: 2013-02-06
url: /2013/02/how-to-handle-a-super-bowl-size-spike-in-web-traffic/
dsq_thread_id:
  - 1068054236
categories:
  - Programming
tags:
  - architecture
  - Http
  - performance
  - scalability
  - web
---
I was shocked to learn the number of [sites which failed to handle the spike in web traffic during the Super Bowl][1]. Most of these sites served static content and should have scaled easily with the use of CDNs. Scaling sites, even dynamic ones, are achievable with well known tools and techniques.

## The Problem is Simple

At a basic level accessing a web page is when one computer, the client, connects to a server and downloads some content. A problem occurs when the number of people requesting content exceeds the ability to deliver content. It&#8217;s just like a restaurant. When there are too many customers people must wait to be served. Staff becomes stressed and strained. Computers are the same. Excessive load causes things to break down.

## Optimization Comes in Three Forms

To handle more requests there are three things you can do: produce (render) content faster, deliver (download) content faster and add more servers to handle more connections. Each of these solutions has a limit. Designing for these limits is architecting for scale.

A page is composed of different types of content: html, css and js. This content is either dynamic (changes frequently) or static (changes infrequently). Static content is easier to scale because you create it once and deliver it repeatedly. The work of rendering is eliminated. Static content can be pushed out to CDNs or cached locally to avoid redownloading. Requests to origin servers are reduced or eliminated. You can also download content faster with small payload sizes. There is less to deliver if there is less markup and the content is compressed. Less to deliver means faster download.

Dynamic content is trickier to cache because it is always changing. Reuse is difficult because pages must be regenerated for specific users at specific times. Scaling dynamic content involves database tuning, server side caching, and code optimization. If you can render a page quickly you can deliver more pages because the server can move on to new requests. Most often, at scale, you want to treat treat dynamic content like static content as best you can.

Adding more servers is usually the easiest way to scale but breaks down quickly. The more servers you have the more you need to keep in sync and manage. You may be able to add more web servers, but those web servers must connect to database servers. Even powerful database servers can only handle so many connections and adding multiple database servers is complicated. You may be able to add specific types of servers, like cache servers, to achieve the results you need without increasing your entire topology.

The more servers you have the harder it is to keep content fresh. You may feel increasing your servers will increase your load. It will become expensive to both manage and run. You may be able to achieve a similar result if you cut your response times which also gives the end user a better experience. If you understand the knobs and dials of your system you can tune properly.

## Make Assumptions

Don&#8217;t be afraid to make assumptions about your traffic patterns. This will help you optimize for your particular situation. For most publicly facing websites traffic is anonymous. This is particularly true during spikes like the Super Bowl. Because you can deliver the same page to every anonymous user you effectively have static content for those users. Cache controls determine how long content is valid and powers HTTP accelerators and CDNs for distribution. You don&#8217;t need to optimize for everyone; split your user base into groups and optimize for the majority. Even laxing cache rules on pages to a minute can shift the burden away from your application servers freeing valuable resources. Anonymous users will get the benefit of cached content with a quick download, dynamic users will have fast servers.

You can also create specific rendering pipelines for anonymous and known users for highly dynamic content. If you can identify anonymous users early you may be able to avoid costly database queries, external API calls or page renders.

## Understand HTTP

HTTP powers the web. The better you understand HTTP the better you can leverage tools for optimizing the web. Specifically look at [http cache headers][2] which allow you to use web accelerators like Varnish and CDNs. The vary header will allow you to split anonymous and known users giving you fine grained control on who gets what. Expiration headers determine content freshness. The worst thing you can do is set cache headers to private on static content preventing browsers from caching locally.

## Try Varnish and ESI

[Varnish][3] is an HTTP accelerator. It caches dynamic content produced from your website for efficient delivery. Web frameworks usually have their own features for caching content, but Varnish allows you to bypass your application stack completely for faster response times. You can deliver a pre-rendered dynamic page as if it were a static page sitting in memory for a greater number of connections.

Edge Side Includes allow you to mix static and dynamic content together. If a page is 90% similar for everyone, you can cache the 90% in Varnish and have your application server deliver the other 10%. This greatly reduces the work your app server needs to do. ESI&#8217;s are just emerging into web frameworks. It will play a more prominent role in Rails 4.

## Use a CDN and Multiple Data Centers

You don&#8217;t need to add more servers to your own data center. You can leverage the web to fan work out across the Internet. I talk more about CDN&#8217;s, the importance of edge locations and latency in my post [Building for the Web: Understanding the Network][4].

Your application servers should be reserved for doing application-specific work which is unique to every request. There are more efficient ways of delivering the same content to multiple people than processing a request top-to-bottom via a web framework. Remember &#8220;the same&#8221; doesn&#8217;t mean the same indefinitely; it&#8217;s the same for whatever timeframe you specify.

If you run Varnish servers in multiple data centers you can effectively create your own CDN. Your database and content may be on the east coast but if you run a Varnish server on the west coast an anonymous user in San Fransisco will have the benefit of a fast response time and you&#8217;ve saved a connection to your app server. Even if Varnish has to deliver 10% dynamic content via an ESI on the east coast it can leverage the fast connection between data centers. This is much better then the end user hoping coast-to-coast themselves for an entire page.

Amazon&#8217;s Route 53 offers the ability to route requests to an optimal location. There are other geo-aware DNS solutions. If you have a multi-region setup you are not only building for resiliency your are horizontally scaling your requests across data centers. At massive scale even load balancers may become overloaded so round-robin via DNS becomes essential. DNS may be a bottleneck as well. If your DNS provider can&#8217;t handle the flood of requests trying to map your URL to your IP address nobody can even get to your data center!

## Use Auto Scaling Groups or Alerting

If you can take an action when things get rough you can better handle spikes. Auto scaling groups are a great feature of AWS when some threshold is maxed. If you&#8217;re not on AWS good monitoring tools will help you take action when things hit a danger zone. If you design your application with auto-scaling in mind, leveraging load balancers for internal communication and avoiding state, you are in a better position to deal with traffic growth. Scaling on demand saves money as you don&#8217;t need to run all your servers all the time. Pinterest gave a talk explaining how it saves money by reducing its server farm at night when traffic is low.

## Compress and Serialized Data Across the Wire

Page sizes can be greatly reduced if you enable compression. Web traffic is mostly text which is easily compressible. A 100kb page is a lot faster to download than a 1mb page. Don&#8217;t forget about internal communication as well. In todays API driven world using efficient serialization protocols like protocol buffers can greatly reduce network traffic. Most RPC tools support some form of optimal serialization. SOAP was the rage in the early 2000s but XML is one of the worst ways to serialize data for speed. Compressed content allows you to store more in cache and reduces network I/O as well.

## Shut Down Features

A performance bottleneck may be caused by one particular feature. When developing new features, especially on a high traffic site, the ability to shut down a misbehaving feature could be the quick solution to a bad problem. Most high-traffic websites &#8220;leak&#8221; new features by deploying them to only 10% of their users to monitor behavior. Once everything is okay they activate the feature everywhere. Similar to determining page freshness for caches, determining available features under load can keep a site alive. What&#8217;s more important: one specific feature or the entire system?

## Non-Blocking I/O

Asynchronous programming is a challenge and probably a last-resort for scaling. Sometimes servers break down without any visible threshold. You may have seen a slow request but memory, cpu, and network levels are all okay. This scenario is usually caused by blocking threads waiting on some form of I/O. Blocked threads are plugs that clog your application. They do nothing and prevent other things from happening. If you call external web services, run long database queries or perform disk I/O beware of synchronous operations. They are bottlenecks. Asynchronous based frameworks like node.js put asynchronous programming at the forefront of development making them attractive for handling numerous concurrent connections. Asynchronous programming also paves the way for queue-based architectures. If every request is routed through a queue and processed by a worker the queue will help even out spikes in traffic. The queue size will also determine how many workers you need. It may be trickier to code but it&#8217;s how things scale.

## Think at Scale

When dealing with a high-load environment nothing can be off the table. What works for a few thousand users will grow out of control for a few million. Even small issues will become exponentially problematic.

Scaling isn&#8217;t just about the tools to deal with load. It&#8217;s about the decisions you make on how your application behaves. The most important thing is determining page freshness for users. The decisions for an up-to-the-second experience for every user are a lot different than an up-to-the-minute experience for anonymous users. When dealing with millions of concurrent requests one will involve a lot of engineering complexity and the other can be solved quickly.

 [1]: http://www.yottaa.com/blog/bid/265815/Coke-SodaStream-the-13-Websites-That-Crashed-During-Super-Bowl-2013
 [2]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html
 [3]: http://www.varnish-cache.org
 [4]: http://www.michaelhamrah.com/blog/2012/01/building-for-the-web-understanding-the-network/
