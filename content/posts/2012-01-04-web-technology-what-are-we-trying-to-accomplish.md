---
title: 'Building for the Web: What Are We Trying to Accomplish?'
author: Michael
type: post
date: 2012-01-04
url: /2012/01/web-technology-what-are-we-trying-to-accomplish/
dsq_thread_id:
  - 527298707
categories:
  - Programming
tags:
  - architecture
  - optimization
  - scalability
  - technology
  - web
---
The web technology landscape is huge and growing every day. There are hundreds of options from servers to languages to frameworks for building the next big thing. Is it [nginx][1] + [unicorn][2] + [rubinus][3] or a [node.js][4] restful service on [cassandra][5] running with [ember.js][6] and html5 on the front end? Should I learn or ? What&#8217;s the best nosql database for a socially powered group buying predicative analysis real-time boutique mobile aggregator that scales to 100 million users and never fails?

It is true there are many choices out there but web technology boils down to a very simple premise. You want to respond to a user action as quickly as possible, under any circumstance, while easily changing functionality. Everything from the technology behind this blog to what goes into facebook operates on that simple idea. The problem comes down to doing it at the scale and speed of the modern web. You have hundreds of thousands of users; you have hundreds of thousands of things you want them to see; you want them to buy, share, create, and/or change those things; you want to deliver a beautiful, customized experience; everything that happens needs to be instantaneous; it can never stop working; and the cherry on the cake is that everything is constantly changing; different features, different experiences, different content; different users. The simple &#8220;hello world&#8221; app is easy. But how do you automatically translate that into every language with a personal message and show a real-time graph with historical data of every user accessing the page _at scale_? What if we wanted to have users leave messages on the same page? Hopefully by the end of this series you will get a sense of how all the pieces fit together and what&#8217;s involved from going to 10 users to 10 thousand to 10 million.

## The Web at 50,000 Feet

The web breaks down to three distinct areas: what happens with the client, what happens on the server, and what happens in between. Usually this is rendering a web page: a user clicks a link, a page delivered, the browser displays the content. It could also involve handling an ajax request, calling an API, posting a search form. Either way it boils down to _client action, server action, result_. Doing this within a users attention span given any amount of meaningful content in a fail-safe way with even the smallest amount of variation is why we have all this technology. It all works together to combine flexibility and speed with power and simplicity. The trick is using the right tool in the swiss-army knife of tech to get the job done.

Let&#8217;s break all this down a bit. Handling a user action well comes down to:

  * Having the server-side handle the request quickly (Serving)
  * A quick travel time between the client and server (Delivering)
  * Quickly displaying the result (Rendering)

And developing and managing all this successfully comes down to:

  * Changing any aspect of what is going on quickly and easily (Developing)

As sites grow and come under heavier load doing any one of these things becomes increasingly difficult. More features, more users, more servers, more code. There is only so much one server can do. There is also only so much a server _needs_ to do. Why hit the database if you can cache the result? Why render a page if you don&#8217;t need to? Why download a one meg html page when it&#8217;s only 100kb compressed? Why download a page if you don&#8217;t have to? How do you do all this and keep your code simple? How do you ensure everything still works even if your [data center goes down][7]?

> Http plays an important role in all of this. I didn&#8217;t truly appreciate http until I read [Restful Web Services][8] by Sam Ruby and Leonard Richardson. Http as an application protocol offers an elegant, scalable mechanism for transferring data and defining intent. Understanding http verbs, various http headers and how http sits on top of tcp/ip can go along way in mastering the web.

## Making it all work together

So how do you choose and use all the tools out there to serve, deliver, render and develop for the web? What does _client action/server action/result_ have to do with [rack][9] and [wsgi][10]? I naïvely thought I could write everything I wanted to in a single post: from using [sass][11] for compressed, minimized css to sharding databases for horizontal scalability. It will be easier to spread it out a bit so stay tuned. But remember: any language, framework or tool out there is really about improving _client action/server action/result_. Even something like [websockets][12]. Websockets eliminates the client request completely. Why wait for a user to tell you something when you can push them content? Knowing your problem domain, your bottlenecks, and your available options will help you choose the right tool and make the right time/cost/benefit decision.

I&#8217;ll dig into the constraints and various techniques to effectively [deliver][13], [serve, develop][14] and [render][15] for the web in upcoming posts.

 [1]: http://www.nginx.org/en/
 [2]: http://unicorn.bogomips.org/
 [3]: http://rubini.us/
 [4]: http://nodejs.org/
 [5]: http://cassandra.apache.org/
 [6]: http://emberjs.com/
 [7]: http://aws.amazon.com/message/65648/
 [8]: http://www.amazon.com/Restful-Web-Services-Leonard-Richardson/dp/0596529260
 [9]: http://rack.rubyforge.org/
 [10]: http://en.wikipedia.org/wiki/Web_Server_Gateway_Interface
 [11]: http://sass-lang.com/
 [12]: http://en.wikipedia.org/wiki/WebSocket
 [13]: http://wp.me/pnRto-af
 [14]: http://wp.me/pnRto-al
 [15]: http://wp.me/pnRto-at
