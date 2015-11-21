---
title: My post about image delivery on the Getty Images Blog
author: Michael
type: post
date: 2011-12-08
url: /2011/12/my-post-about-image-delivery-on-the-getty-images-blog/
dsq_thread_id:
  - 527224466
categories:
  - Programming
---
I wrote an article covering how we move images to our customers on the new [Getty Images blog][1].

The system is a suite of .NET applications which handle various steps in our workflow. It features:

  * A custom C# module which sits with IIS FTP to alert when a new image arrives
  * Services built with the [Topshelf framework][2]
  * WCF services which wrap a rules engine featuring dynamic code generation

The Getty Images blog be covering more insight into the system as well as other technology developed at Getty Images. So check out the new blog and subscribe to the RSS feed!

 [1]: http://blog.gettyimages.com/2011/12/06/from-camera-to-customer-faster-than-ever-before/
 [2]: https://github.com/Topshelf/Topshelf
