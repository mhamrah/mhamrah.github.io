---
title: 'Quicktip: Use Negative Margins with CSS Transforms to Fix Clipping'
author: Michael
type: post
date: 2011-03-02
url: /2011/03/quicktip-use-negative-margins-with-css-transforms-to-fix-clipping/
dsq_thread_id:
  - 532823522
categories:
  - Programming
tags:
  - css
  - html
  - quicktip
---
I&#8217;ve been playing around with CSS Transforms and had an annoying issue: when rotating divs at an angle, the edge of the div also rotated leaving a gap where I didn&#8217;t want one. See the pic:

[<img src="http://i2.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2011/03/rotated-div.png?resize=246%2C117" alt="" title="rotated div2" class="aligncenter size-full wp-image-531" data-recalc-dims="1" />][1]

In this case, the div is actually a header tag I wanted to span the length of the page, like a ribbon stretching the width of browser. But I did not want that gap. So what to do? I thought about using Transforms to skew the header the angle required to maintain a vertical line, but that&#8217;s annoying. 

Instead, I simply added a negative margin to the width to stretch the header enough to hide the gap. Here&#8217;s the css and final result:

<pre class="syntax css">header 
{
  background-color: #191919;
  margin: 75px -20px;
  -webkit-transform: rotate(-10deg);
  -moz-transform: rotate(-10deg);
  tranform:rotate(-10deg);
}
</pre>

[<img src="http://i0.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2011/03/rotated-div2.png?resize=246%2C117" alt="" title="rotated div2" class="aligncenter size-full wp-image-531" data-recalc-dims="1" />][2]

You may find rotate and skew is a better combination to achieve the desired result- however, if you have text in your containing element, that text will also be skewed if you use skew. 

To achieve the desired result within a page (where you don&#8217;t have the benefit of viewport clipping) you can always put the rotated element within another element, use negative margins, and set <code class="syntax css">overflow:hidden</code> to achieve the desired result.

 [1]: http://i2.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2011/03/rotated-div.png
 [2]: http://i0.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2011/03/rotated-div2.png
