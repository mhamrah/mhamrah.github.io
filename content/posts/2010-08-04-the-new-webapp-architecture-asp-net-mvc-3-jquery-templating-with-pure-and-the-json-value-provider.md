---
title: 'The New Web App Architecture: ASP.NET MVC 3, jQuery Templating with PURE and the Json Value Provider'
author: Michael
type: post
date: 2010-08-04
url: /2010/08/the-new-webapp-architecture-asp-net-mvc-3-jquery-templating-with-pure-and-the-json-value-provider/
dsq_thread_id:
  - 527224523
categories:
  - Programming
---
Over the past couple of years there has been a slow progression in the .NET web app world to fully separate out client/server interaction.  Long gone are the horrible days of ViewState and Events; MVC provided a nice step to better structure web applications for powerful Web 2.0 experiences.  But the barrier between client and server interaction has never really been clean-  MVC markup has always been littered with C# code and there hasn&#8217;t always been widespread tools available to easily build desktop class applications in the browser.  Sure, spark and haml provide alternatives, but these are essentially make a core problem easier to bear.

With the new preview of MVC 3, we can eliminate that core problem of server based html rendering: the built in Json Support via the JsonValueProviderFactory along with some jQuery goodies presents us with the new web app architecture: service- orientated web apps built with a rich, js based client driven by Json based services.  Using core tools of html, css, and js and not requiring the overhead of Silverlight, Flash, or JS based web frameworks.  This JSON functionality, combined with javascript templating using [PURE][1] gives us the ability to break free from the static html navigation and build dynamic apps in much more efficient ways.

### ASP.NET MVC 3 and the JsonValueProviderFactory

[Scott Guthrie writes about the new ASP.NET MVC 3 preview features][2], and among the hype is the default Json support for action methods.  In his blog post he forgets one crucial step to actually make this work, which is adding the new JsonValueProviderFactory to the global value provider factories class in the Application start method of the global.asax, like so:

<pre class="syntax c#">protected void Application_Start()
{
	AreaRegistration.RegisterAllAreas();

	//Must add this factory explicitly (for now, at least):
	ValueProviderFactories.Factories.Add(new JsonValueProviderFactory());

	RegisterGlobalFilters(GlobalFilters.Filters);
	RegisterRoutes(RouteTable.Routes);
}
</pre>

Once this is in place we can treat our controller actions as usual, even when using Json.  The serialization mechanism is agnostic (this is pretty much exactly the same as Scott&#8217;s code):

<pre class="syntax c#">[HttpPost]
public ActionResult Search(ImageSearchInput input)
{
	//AssetSearchInput can be posted via Json
	return new JsonResult() { Data = new { ImageInfo = new Repository.ImageRepository().Search(input).Images } };
}
</pre>

ImageSearchInput, with a string property of &#8220;Caption&#8221;, will be built from the Json data posted to the server by the following Ajax call:

<pre class="syntax js">$('#search').submit(function () {
  var input = { Caption: $('#caption').val() };

  $.ajax({url: '/Home/Search',
    type: 'POST',
    data: JSON.stringify(input),
    dataType: 'json',
    contentType: 'application/json; charset=utf-8',
    success: function (data) {
        $('.imgContainer').autoRender(data);
    }
  });

  return false;
});
</pre>

The server will route the request to the Home/Search method, and will serialize the posted data in the ImageSearchInput class.  As long the properties match up and are of the correct type, the deserialization will be fluid.  Notice how we don&#8217;t actually need to specify the ImageSearchInput class name when building the Json object.

Those with keen eyes may have noticed the success: callback containing the autoRender() function.  This is the [Over the past couple of years there has been a slow progression in the .NET web app world to fully separate out client/server interaction.  Long gone are the horrible days of ViewState and Events; MVC provided a nice step to better structure web applications for powerful Web 2.0 experiences.  But the barrier between client and server interaction has never really been clean-  MVC markup has always been littered with C# code and there hasn&#8217;t always been widespread tools available to easily build desktop class applications in the browser.  Sure, spark and haml provide alternatives, but these are essentially make a core problem easier to bear.

With the new preview of MVC 3, we can eliminate that core problem of server based html rendering: the built in Json Support via the JsonValueProviderFactory along with some jQuery goodies presents us with the new web app architecture: service- orientated web apps built with a rich, js based client driven by Json based services.  Using core tools of html, css, and js and not requiring the overhead of Silverlight, Flash, or JS based web frameworks.  This JSON functionality, combined with javascript templating using [PURE][1] gives us the ability to break free from the static html navigation and build dynamic apps in much more efficient ways.

### ASP.NET MVC 3 and the JsonValueProviderFactory

[Scott Guthrie writes about the new ASP.NET MVC 3 preview features][2], and among the hype is the default Json support for action methods.  In his blog post he forgets one crucial step to actually make this work, which is adding the new JsonValueProviderFactory to the global value provider factories class in the Application start method of the global.asax, like so:

<pre class="syntax c#">protected void Application_Start()
{
	AreaRegistration.RegisterAllAreas();

	//Must add this factory explicitly (for now, at least):
	ValueProviderFactories.Factories.Add(new JsonValueProviderFactory());

	RegisterGlobalFilters(GlobalFilters.Filters);
	RegisterRoutes(RouteTable.Routes);
}
</pre>

Once this is in place we can treat our controller actions as usual, even when using Json.  The serialization mechanism is agnostic (this is pretty much exactly the same as Scott&#8217;s code):

<pre class="syntax c#">[HttpPost]
public ActionResult Search(ImageSearchInput input)
{
	//AssetSearchInput can be posted via Json
	return new JsonResult() { Data = new { ImageInfo = new Repository.ImageRepository().Search(input).Images } };
}
</pre>

ImageSearchInput, with a string property of &#8220;Caption&#8221;, will be built from the Json data posted to the server by the following Ajax call:

<pre class="syntax js">$('#search').submit(function () {
  var input = { Caption: $('#caption').val() };

  $.ajax({url: '/Home/Search',
    type: 'POST',
    data: JSON.stringify(input),
    dataType: 'json',
    contentType: 'application/json; charset=utf-8',
    success: function (data) {
        $('.imgContainer').autoRender(data);
    }
  });

  return false;
});
</pre>

The server will route the request to the Home/Search method, and will serialize the posted data in the ImageSearchInput class.  As long the properties match up and are of the correct type, the deserialization will be fluid.  Notice how we don&#8217;t actually need to specify the ImageSearchInput class name when building the Json object.

Those with keen eyes may have noticed the success: callback containing the autoRender() function.  This is the][3] javascript templating engine at work.  I came across PURE when reading about the [jQuery templating proposal on GitHub][4] and was immediately drawn to its simplistic syntax.

The philosophy behind PURE is simple: instead of interleaving markup and template directives (which, to be honest, is just as bad as mixing code with markup), PURE can make assumptions about what to repeat and what to bind based on css class names and Json property names.

Let&#8217;s say I had the following json markup:

<pre class="syntax js">var data = {
	Image:
		{ Filename: 'mypic1.jpg', ImageUrl: '/images/mypic1.jpg'},
		{ Filename: 'agoodphoto.jpg', ImageUrl: '/images/agoodphoto.jpg'}
}
</pre>

This is simply an Array with two objects in it.  Using PURE&#8217;s autoRender function, we can specify binding directives using CSS classes, building li elements and populating content as appropriate:

<pre class="syntax html escaped">&lt;ul class="imgContainer"&gt;
	&lt;li class="Image"&gt;
		&lt;p&gt;&lt;img class="ImageUrl@src" /&gt;&lt;/p&gt;
		&lt;p class="info"&gt;&lt;span class="Filename"&gt;&lt;/span&gt;&lt;/p&gt;
	&lt;/li&gt;
&lt;/ul&gt;
</pre>

Because the li has the Image css class, and we have an array of Image objects, PURE will duplicate the li contents for each element in the array.  The span tag will show the filename property because it has the Filename css class.  The src attribute of the img tag will get the ImageUrl property because it is using the @ directive, which simply says &#8220;put the ImageUrl property in the src attribute&#8221;.  PURE can easily get it&#8217;s own post, but the point is simple:  we no longer need to generate server side html for displaying complex content.  Even though this has always been possible before, it hasn&#8217;t been as fluid as PURE.  Another primary benefit is that we can send incredibly complex Json data back to the client which can update numerous page snippets.  Orchestrating multi-updates, like a detail page, shopping cart, or other areas simultaneously has not been trivial, but because we have json data and templating these complex scenarios are easily achievable.

### Gotchas

Client side templating is dangerous- you don&#8217;t really know how well the client&#8217;s computer it is.  Be mindful of performance and memory requirements, and ensure you&#8217;re targeting the right browsers.  Chrome&#8217;s V8 javascript engine has evolved remarkably well in handling javascript intensive applications.

As for the MVC 3 preview, I wish we saw dynamic results from controllers: specifically, one action to return either  Json, Xml, or Html output based on some request directive.  This is possible with various ActionResults or ActionFilters, and there&#8217;s some functionality already out there to do it, but it isn&#8217;t as nice as Ruby on Rail&#8217;s respond_to functionality.  A single action supporting multiple outputs will allow developers to easily target various platforms and scenarios in the web world.

### Final Thoughts

Despite techniques of Json serialization and jQuery templating already existing in the ecosystem, it has never been as robust and integrated as with the new MVC 3 support (sure, Rails has had this for a while).  Combining this functionality with templating tools like PURE will open up new development paths for rich web applications based entirely on standard tools like javascript, html, and css without relying on chunky js frameworks.  Keeping core tools simple means having the ability to build out specific functionality as needed, rather than getting boxed into more complete frameworks.  From the managing side, it also keeps the available pool of developers large so you&#8217;re not requiring knowledge on a specific tool.  Play around with these tools and see what you can do!
  
  

  
<a rev="vote-for" href="http://dotnetshoutout.com/The-New-Web-App-Architecture-ASPNET-MVC-3-jQuery-Templating-with-PURE-and-the-Json-Value-Provider-Adventures-in-HttpContext"><img alt="Shout it" src="http://dotnetshoutout.com/image.axd?url=http%3A%2F%2Fwww.michaelhamrah.com%2Fblog%2F2010%2F08%2Fthe-new-webapp-architecture-asp-net-mvc-3-jquery-templating-with-pure-and-the-json-value-provider%2F" style="border:0px" /></a>

 [1]: http://beebole.com/pure/
 [2]: http://weblogs.asp.net/scottgu/archive/2010/07/27/introducing-asp-net-mvc-3-preview-1.aspx
 [3]: http://beebole.com/pure
 [4]: http://wiki.github.com/nje/jquery/jquery-templates-proposal
