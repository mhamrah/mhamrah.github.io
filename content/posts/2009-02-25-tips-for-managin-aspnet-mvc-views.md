---
title: Tips for Managing ASP.NET MVC Views
author: Michael
type: post
date: 2009-02-25
url: /2009/02/tips-for-managin-aspnet-mvc-views/
dsq_thread_id:
  - 527224610
categories:
  - Programming
tags:
  - aspnetmvc
---
After working on an ongoing ASP.NET MVC project for a couple of months I&#8217;ve learned a couple of lessons when it comes to dealing with Views.  Keep as much logic out of the views as possible! This can be tricky because it&#8217;s so easy to let code sneak into your views. But by following the tips below you&#8217;ll be able to keep your logic organized and views clean.

**1) Follow the Rails pattern of having a Helper class for each controller.  The helper class deals with html snippets or formatting functions on a per controller level.**

[<img class="alignnone size-medium wp-image-174" title="ASP.NET MVC Helpers" src="http://i1.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2009/02/helpersfolder.png?resize=211%2C159" alt="ASP.NET MVC Helpers" data-recalc-dims="1" />][1]

Pushing out bits of code like date formatting into helper classes not only cleans up the views, but aids in testability.  Logic is consolidated in one place simplifying maintenance.  Creating Helpers on a per controller level also creates a namespace where functionality can be found intuitively.
  
**2) Don&#8217;t overload the Html helper with one-off functions. Organize functions into their respective helper classes.**

It&#8217;s so easy to want to add extension methods to the HtmlHelper class.  It&#8217;s like a siren crying out &#8220;do it, do it&#8221;.  This is bad!  Extension methods are good for some things, but can easily garble an API.  The HtmlHelper class should be reserved for rendering core html elements.  As an alternative, leverage helper classes you create on a per-controller basis.

<pre class="syntax c#">//These shouldn't belong to HtmlHelper!
public static string ShowSomethingOnlyForHomeController((this HtmlHelper helper) ...
public static string RenderDateTimeNowInSpan(this HtmlHelper helper) ...</pre>

**3) Leverage partial views for iterative content, even if it&#8217;s not reused elsewhere (hint: it probably will be eventually).** 

So instead of:

<pre class="syntax c#">&lt;%= foreach (var photo in photos)
{ %>


<div class="photo">
  <span>&lt;%= photo.Name %></span>
  
  
  <p>
    &lt;%= photo.Description %>
  </p>
  
  
  <p>
    &lt;%= photo.Date %>
  </p>
  
</div>
&lt;%= } %>
</pre>

use:

<pre class="syntax c#">&lt;% foreach (var photo in photos)
{ %>
&lt;% Html.RenderPartial("~/Views/Photos/PhotoTemplate", photo); %>
&lt;% } %>
</pre>

This will greatly minimize the amount of markup scattered throughout a view, keeping view files focused on a specific task (just like good class design).  It&#8217;ll also make version control easier because changes will be isolated to their respective file, allowing someone to better see what changes happened where.  It also eliminates excessive merging.

**4) Organize partial views within their respective controller, not a shared folder- even if it&#8217;s a global skin.** 

Dumping partials within a shared folder can cause overcrowding and jumbling in the long term.  Prefer organization and grouping of related content (again, just like good class design).
  
**5) Prefer a strongly typed view and leverage specialized ViewData types for aggregating random models under one root.** 

It can be easy to dump stuff into the ViewData hash.  However, prefer using a custom ViewData class instead.  It&#8217;s easier to know what data is available for the view.  This is a lot more intuitive than rummaging through a controller, which happens when teams share code.  Also leverage the [null object pattern][2] for properties to avoid having to do null reference checks in the views.

Instead of:

<pre class="syntax c#">ViewData["Title"] = "My Photos";
ViewData["Photos"] = myPhotos;
ViewData["User"] = currentUser;

return View();
</pre>

use:

<pre class="syntax c#">//Title and User can be properties of a base view data class.
var vd = new PhotoListViewData() 
     { Photos = myPhotos, Title = "My Photos", User = currentUser };
return  View(vd);

//Sample null object pattern (always returns a valid object, so no if null or Count == 0):
private List&lt;Photo> _photos = new List&lt;Photo>();
public List&lt;Photo> Photos 
{ 
get 
     { if (_photos == null) _photos = new List&lt;Photo>(); 
            return _photos; 
     } 
set { 
        _photos = value; } 
}
</pre>

**6) Minimize code snippets in views.** 

Code snippets occur in many ways.  They can be as simple as formatting a date, to string concatenation, to even doing a grouping/projection to get data correct.  Doing this in a view easily leads to bugs and isn&#8217;t testable.  Any data processing specifically for views should occur in the controller action, custom ViewData object itself, or in a helper class.  Code in views should be simplified to looping, calling a property, or calling a single method.  Anything more than that will get you into trouble!
  
**Leverage RenderAction or MvcContrib&#8217;s SubController for dealing with shared isolated functionality not relevant to a view.**

Unfortunately, as of RC1 there still isn&#8217;t a great way to deal with aggregating disparate content in an action.  You&#8217;ll need to resort to the RenderAction in the Future&#8217;s dll or use MvcContrib&#8217;s SubController.  The point is the same- keep actions specific to what you&#8217;re doing.  If you need to aggregate disparate content in a view (like a menu in the header, or a shopping cart) offload functionality into an action and call RenderAction.  Having actions do multiple, random things leads to messy code.  Prefer a single point of entry into supporting view content.

Good luck and share your tips!

[<img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f02%2ftips-for-managin-aspnet-mvc-views%2f&bgcolor=0000FF" border="0" alt="kick it on DotNetKicks.com" />][3]

 [1]: http://i1.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2009/02/helpersfolder.png
 [2]: http://en.wikipedia.org/wiki/Null_Object_pattern
 [3]: http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f02%2ftips-for-managin-aspnet-mvc-views%2f
