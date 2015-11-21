---
title: Using Flickr and jQuery to learn JSONP
author: Michael
type: post
date: 2010-02-05
url: /2010/02/using-flickr-and-jquery-to-learn-jsonp/
dsq_thread_id:
  - 528857062
categories:
  - Programming
tags:
  - jquery
---
I was playing around with the Flickr API recently and got a little stuck: when using jQuery to call the [Flickr public feed][1] using jQuery&#8217;s $.getJSON method, I wasn&#8217;t getting any results.  I thought maybe I was parsing the response incorrectly, but when I went to check out the data coming back in firebug, nothing was there.  I couldn&#8217;t believe it- the response headers were present, but the body was blank.  Calling the public feed url from the browser worked fine.  What&#8217;s more interesting was everything worked in IE.  So I did some experimenting and learned the issue: I wasn&#8217;t correctly using the endpoint to work with [JSONP][2], which is required when using jQuery with Flickr.  Then I thought I better learn more about JSONP.

There are [plenty of good articles][3] about [JSONP][4] on the net.  Essentially, JSONP allows you to specify custom callbacks when making remote ajax calls.  Firefox seems to be more strict when dealing with jsonp, which is why I didn&#8217;t get a response body.  What did the trick was adding the

<pre class="syntax js">jsoncallback=?</pre>

query string parameter to the end of my Flickr url.  This allows the jQuery framework to route to the default success function you pass to .getJSON(), like so:

<pre class="syntax js">$.getJSON(
            'http://api.flickr.com/services/feeds/photos_public.gne?format=json&jsoncallback=?',
               function(data) {
                   $.each(data.items, function(i, item) {
                       $("img").attr("src", item.media.m).appendTo("#images");
                   });
               });
</pre>

[Here&#8217;s an example of this call][5].   So what happens if we replace the ? with something else? Well, passing a simple text string will be treated like a normal function call.  Take a look at the example below:

<pre class="syntax js">function GetItemsUsingExternalCallback() {
            $.ajax({ url: 'http://api.flickr.com/services/feeds/photos_public.gne?format=json&jsoncallback=myCallbackFunction',
                dataType: 'jsonp'
            });
        }
  function myCallbackFunction(data) {
  //dostuff
  }
</pre>

jQuery will try and call &#8220;myCallbackFunction&#8221; as a normal function call instead of routing to the standard success function that&#8217;s part of the getJSON call.  [The example page also includes this approach.][5] There&#8217;s only a slight difference between the two approaches, but it&#8217;s cool that jQuery can call a function outside of the normal success callback.  Of course, if you needed to reuse a common callback across multiple ajax calls, you&#8217;d probably want to call that function directly from the success method, rather than inlining the function in the .ajax call.

 [1]: http://www.flickr.com/services/feeds/
 [2]: http://bob.pythonmac.org/archives/2005/12/05/remote-json-jsonp/
 [3]: http://www.insideria.com/2009/03/what-in-the-heck-is-jsonp-and.html
 [4]: http://en.wikipedia.org/wiki/JSON#JSONP
 [5]: http://www.michaelhamrah.com/blog/wp-content/FlickrJsonp.html
