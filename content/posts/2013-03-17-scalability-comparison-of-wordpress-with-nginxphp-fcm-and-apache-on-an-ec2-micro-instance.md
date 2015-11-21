---
title: Scalability comparison of WordPress with NGINX/PHP-FCM and Apache on an ec2-micro instance.
author: Michael
type: post
date: 2013-03-17
url: /2013/03/scalability-comparison-of-wordpress-with-nginxphp-fcm-and-apache-on-an-ec2-micro-instance/
dsq_thread_id:
  - 1145297149
categories:
  - Programming
tags:
  - apache
  - architecture
  - nginx
  - performance
  - php
  - scalability
---
For the past few years this blog ran apache + mod_php on an ec2-micro instance. It was time for a change; I&#8217;ve enjoyed using nginx in other projects and thought I could get more out of my micro server. I went with a php-fpm/nginx combo and am very surprised with the results. The performance charts are below; for php the response times varied little under minimal load, but nginx handled heavy load far better than apache. Overall throughput with nginx was phenomenal from this tiny server. The result for static content was even more impressive: apache effectively died after ~2000 concurrent connections and 35k total pages killing the server; nginx handled the load to 10,000 very well and delivered 160k successful responses.

Here&#8217;s the [loader.io][1] results from static content from http://www.michaelhamrah.com, comparing apache with nginx. I suggest clicking through and exploring the charts:

<div style="width: 600px;">
  </p> 
  
  <div style="width: 100%; text-align: right;">
    <a href="http://loader.io/results/f1c357b13b1f554eef534b79866eb5ce" target="_blank"  style="padding: 0 10px 10px 0; font-family: Arial, 'Helvetica Neue', Helvetica, sans-serif; font-size: 14px;">View on loader.io</a>
  </div>
</div>

Apache only handled 33.5k successful responses up to about 1,300 concurrent connections, and died pretty quickly. Nginx did far better:

<div style="width: 600px;">
  </p> 
  
  <div style="width: 100%; text-align: right;">
    <a href="http://loader.io/results/9430bdfcab50f31dc66f3ea3014beb84" target="_blank"  style="padding: 0 10px 10px 0; font-family: Arial, 'Helvetica Neue', Helvetica, sans-serif; font-size: 14px;">View on loader.io</a>
  </div>
</div>

160k successful response with a 22% error rate and avg. response time of 142ms. Not too shabby. The apache run effectively killed the server and required a full reboot as ssh was unresponsive. Nginx barely hiccuped.

The results of my wordpress/php performance is also interesting. I only did 1000 concurrent users hitting blog.michaelhamrah.com. Here&#8217;s the apache result:

<div style="width: 600px;">
  </p> 
  
  <div style="width: 100%; text-align: right;">
    <a href="http://loader.io/results/210867953c97cdd2dd4308dce17bcae3" target="_blank"  style="padding: 0 10px 10px 0; font-family: Arial, 'Helvetica Neue', Helvetica, sans-serif; font-size: 14px;">View on loader.io</a>
  </div>
</div>

There was a 21% error rate with 13.7k request served and a 237ms average response time (I believe the lower average is due to errors). Overall not too bad for an ec2-micro instance, but the error rate was quite high and nginx again did far better:

<div style="width: 600px;">
  </p> 
  
  <div style="width: 100%; text-align: right;">
    <a href="http://loader.io/results/631e11ff9206c6c7a3820c891380c9a3" target="_blank"  style="padding: 0 10px 10px 0; font-family: Arial, 'Helvetica Neue', Helvetica, sans-serif; font-size: 14px;">View on loader.io</a>
  </div>
</div>

A total of 19k successes with a 0% error rate. The average response time was a little higher than apache, but nginx did serve far more responses. I also get a kick out of the response time line between the two charts. Apache is fairly choppy as it scales up, while nginx increases smoothly and evens out when the concurrent connections plateaus. That&#8217;s what scalability should look like!

There are plenty of guides online showing how to get set up with nginx/php-fpm. [The Nginx guide on WordPress Codex][2] is the most thorough, but there&#8217;s a [straightforward nginx/php guide on Tod Sul][3]. I also relied on an [nginx tuning guide from Dakini][4] and [this nginx/wordpress tuning guide from perfplanet][5]. They both have excellent information. I also think you should check out the [html5 boilerplate nginx conf files][6] which have great bits of information.

If you&#8217;re setting this up yourself, start simple and work your way up. The guides above have varying degrees of information and various configuration options which may conflict with each other. Here&#8217;s some tips:

  1. Decide if you&#8217;re going with a socket or tcp/ip connection between nginx + php-fcm. A socket connection is slightly faster and local to the system, but a tcp/ip is (marginally) easier to set up and good if you are spanning multiple nodes (you could create a php app farm to compliment an nginx front-facing web farm).
  
    I chose to go with the socket approach between nginx/php-fpm. It was relatively painless, but I did hit a snag passing nginx requests to php. I kept getting a &#8220;no input file specified&#8221; error. It turns out it was a simple permissions issue: the default php-fpm user was different the nginx user the webserver runs under. Which leads me to:
  2. Plan your users. Security issues are annoying, so make sure file and app permissions are all in sync.
  3. Check your settings! Read through default configuration options so you know what&#8217;s going on. For instance you may end up running more worker processes in your nginx instance than available cpu&#8217;s killing performance. Well documented configuration files are essential to tuning.
  4. Plan for access and error logging. If things go wrong during the setup, you&#8217;ll want to know what&#8217;s going on and if your server is getting requests. You can turn access logs of later.
  5. Get your app running, test, and tune. If you do too many configuration settings at once you&#8217;ll most likely hit a snag. I only did a moderate amount of tuning; nginx configuration files vary considerably, so again it&#8217;s a good idea to read through the options and make your own call. Ditto for php-fcm.

I am really happy with the idea of running php as a separate process. Running php as a daemon has many benefits: you have a dedicate process you can monitor and recycle for php without effecting your web server. Pooling apps allows you to tune them individually. You&#8217;re also not tying yourself to a particular web server; php-fpm can run fine with apache. In TCP mode you can even offload your web server to separate node. At the very least, you can distinguish php usage against web server usage.

So my only question is why would anyone still use apache?

 [1]: http://loader.io
 [2]: http://codex.wordpress.org/Nginx
 [3]: http://todsul.com/install-configure-php-fpm
 [4]: http://dak1n1.com/blog/12-nginx-performance-tuning
 [5]: http://calendar.perfplanet.com/2012/using-nginx-php-fpmapc-and-varnish-to-make-wordpress-websites-fly/
 [6]: https://github.com/h5bp/server-configs/blob/master/nginx/nginx.conf
