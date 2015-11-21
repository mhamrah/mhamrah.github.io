---
title: Wiping Out Inherited CSS Styles
author: Michael
type: post
date: 2009-02-27
url: /2009/02/wiping-out-inherited-css-styles/
dsq_thread_id:
  - 527224629
categories:
  - Programming
tags:
  - css
---
Sure, you can use the !important css keyword in your stylesheets to override inherited styles, but what about when you want to simply wipe out or remove a style defined in a parent? Use the auto keyword.

Let&#8217;s say you have a style defined in a global.css sheet defining a position for a class called menu:

<pre class="brush: css; title: ; notranslate" title="">ul.menu {
position:absolute;
left: 10px;
top: 10px;
}
</pre>

This will position a <ul class=&#8221;menu&#8221;> element</ul> 10 pixels away from the top left corner of the parent element.  But what if you want to move that menu to the right side of the parent element, but you can&#8217;t change the defined style in global.css?

You create a new stylesheet or define an inline style, and override the ul.menu class by specifying

<pre class="brush: css; title: ; notranslate" title="">ul.menu {
right:10px;
}
</pre>

But this simply stretches the menu to the other corner.   We need to override the left:10px; style with the default style (as if we never specified the left:10px; to begin with).  left:none; won&#8217;t work and left:0; won&#8217;t either.  The solution is the auto keyword:

<pre class="brush: css; title: ; notranslate" title="">ul.menu {
right:10px;
left:auto; /* removes left:10px from parent */
}
</pre>

Voila, you&#8217;re done!  It&#8217;s as if the left:10px was never defined!
