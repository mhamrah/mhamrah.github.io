---
title: 'Building for the Web: Understanding The Network'
author: Michael
type: post
date: 2012-01-06
url: /2012/01/building-for-the-web-understanding-the-network/
dsq_thread_id:
  - 529230134
categories:
  - Programming
tags:
  - architecture
  - network
  - optimization
  - scalability
  - technology
  - web
---
My [first post on web technology][1] talks about what we are trying to accomplish when building for the web. There are four ways we can break down the standard flow of _client action/server action/result_: delivering, serving, rendering and developing. This post focuses on delivering content by understanding the network. Why use a [cdn][2]? What&#8217;s all the fuss about connections and compressed static assets? The network is often overlooked but understanding how it operates is essential for building high performing websites. A 50ms rendering time with a 50ms db query is meaningless if it takes three seconds to download a page.

## TCP: Know It.

Going from client to server and back again rests on the network and how well you use it. [TCP][3] dominates communication on the web and is worth knowing well. In order to send data from one point to another a connection is established between two points via a back-and-forth handshake. Once established, data flows between the two in a series of packets. TCP offers reliability by acknowledging receipt of every packet sent by sending a second acknowledgement packet back to the server. The time it takes to go from one end to another and back is called latency or round-trip time. At any given time there are packets in flight waiting acknowledgement of receipt. TCP only allows a certain amount of unacknowledged packets in flight; this is called window size. Connections start with small window sizes but as more successful transfers occur the window size will increase ([known as slow start][4]). This effectively increases bandwidth because more data is sent at once. The longer the latency the slower a connection; if the window size is full then the server must wait for acknowledgements before sending more data. This is on top of the time it actually takes to send packets. The best case scenario is low latency with large, full windows.

## Reliability

Reliable connections are also important; if packets are lost they must be resent. Retransmissions slow down transfers because tcp guarantees in-order delivery of data to the application layer. If a packet is dropped and needs to be resent nothing will be delivered to the application until that one packet is received. UDP, an alternative to TCP, doesn&#8217;t offer the same guarantees and assumes issues are dealt with at the application layer.

There is a [good article by Phillip Tellis on understanding the network with JS][5] which talks about data transfer and TCP. [Wireshark][6] is another great tool for analyzing packets across a network. You can actually view individual packets as they come and go and see how window size is scaling, view retransmissions, measure bandwidth, and examine latency.

## The Importance of Connections

Establishing a connection takes time because of the handshake involved and latency considerations. A 100ms latency could mean more than 300ms before any data is even received on top of dealing with any dns lookups and os overhead. Keeping a connection alive avoids this creation overhead. Connection pooling, for example, is a popular technique to manage database connections. A web server will talk to a database frequently; if the connection is already established there is no overhead in executing a new query which can trim valuable time off of serving a request. [Ping][7] and [traceroute][8] are two worthwhile tools that examine latency and the &#8220;hops&#8221; packets take from one network to another as they travel from end to end.

Connections take time to create, have a relatively limited availability and require overhead to manage. It may seem like a great idea to keep connections open for the long haul, but there is a limited number of connections a server can sustain. Concurrent connections is a popular benchmark which examines how many simultaneous connections can occur at any given time. If you hit that mark, new requests will have to wait until something frees up. The ideal situation is to pump as much as possible through a few connections so there are more available to others. If you can serve multiple files files at once great; but why keep six empty connections open between requests if you don&#8217;t need too?

> On a side note, this is where server architecture comes into play. A server usually processes a request by building a web page from a framework. If this can be done asynchronously by the web server, or offloaded somewhere else, the web server can handle a higher number of sustained connections. The server can grab a new connection while it waits for data to send on an existing one. We&#8217;ll talk more about this in another post. Fast server times also keep high concurrent connections from arising in the first place.

## Optimizing the Network

Lowering latency and optimizing data throughput are what dominate delivery optimization. It is important to keep the data which flows between client and server to a minimum. Downloading a 100kb page is a lot faster than downloading a 1000kb page. Compressing static content like css and javascript greatly reduces payload which is why tools like [Jammit][9] and [Google Closure][10] are so ubiquitous. These tools can also merge files; because of http chatter it is faster to download one larger file than several small files. Remember the importance of knowing http? Each http request requires reusing or establishing a connection, a header request sent, the server handling the request, and the response. Doing this once is better than twice. Most web servers can also dynamically compress http responses and should be used when possible.

Fixing latency can be done by using a content delivery network like [Amazon Cloudfront][11] or [Akamai][12]. They shorten the distance between a request and response by taking content from your server and spreading it on their infrastructure all over the world. When a user requests a resource the cdn routes the request to the server with the lowest latency. A user in Japan can download a file from Japan a lot faster than he can from Europe. Shorter distance, fewer hops, fewer retransmissions. A good cdn strategy rests on how easy it is to push content to a cdn and how easy it is to refresh it. Both concerns should be well researched when leveraging a cdn. You don&#8217;t want stale css files in the wild when you release a new version of your app.

A [WAN accelerator][13] is also a cool technique. Let&#8217;s say you want to deliver a dynamic web page from the US to Tokyo. You could have that travel over the open internet with a high latency connection. Or you could route that request to a data center in Tokyo with an optimized connection to the US. The user gets a low-latency connection to the Tokyo datacenter which in turn gets a low-latency high bandwidth connection to the US. This can greatly simplify issues with running and keeping multiple data centers in sync.

## The Bottom Line

There&#8217;s a lot of effort underway in making the web faster by changing how tcp connections are leveraged on the web. Http 1.0 requires a new connection for every request/response and browsers limit the number of parallel connections between client and server between two and six. Http keep-alive and [http pipelining][14] offer mechanisms to push more content through existing connections. Rails 3.1 introduced http streaming via chunked responses. Browsers can fetch assets in parallel with the main html response as soon as tags appear in the main response stream. [Spdy][15], an effort by Google, is worth checking out: it proposes a multi-pronged attack to leverage as much flow as possible on a single connection. The docs also illustrate interesting pain points with the network on the web. The bottom line is simple: reduce the amount of data that needs to go from one place to another and make the travel time as fast as possible. Small amounts of data over existing parallel connections make a fast web.

This approach shouldn&#8217;t be limited to users and servers; optimizing network communication within your datacenter is extremely important. You have total control over your infrastructure and can tune your network accordingly. You can also choose your communication protocols: using something like [thrift][16] or [protocol buffers][17] can save a tremendous amount of bandwidth over xml-based web services on http.</p> 

:http://www.scala-lang.org/</p> 

:http://python.org/

 [1]: http://wp.me/pnRto-aa
 [2]: http://en.wikipedia.org/wiki/Content_delivery_network
 [3]: http://en.wikipedia.org/wiki/Transmission_Control_Protocol
 [4]: http://en.wikipedia.org/wiki/Slow-start
 [5]: http://coding.smashingmagazine.com/2011/11/14/analyzing-network-characteristics-using-javascript-and-the-dom-part-1/
 [6]: http://www.wireshark.org/
 [7]: http://en.wikipedia.org/wiki/Ping
 [8]: http://en.wikipedia.org/wiki/Traceroute
 [9]: http://documentcloud.github.com/jammit/
 [10]: http://code.google.com/closure/
 [11]: http://aws.amazon.com/cloudfront/
 [12]: http://www.akamai.com
 [13]: http://en.wikipedia.org/wiki/WAN_optimization
 [14]: http://en.wikipedia.org/wiki/HTTP_pipelining
 [15]: http://www.chromium.org/spdy/spdy-whitepaper
 [16]: http://thrift.apache.org/
 [17]: http://code.google.com/p/protobuf/
