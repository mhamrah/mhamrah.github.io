---
title: Nina, My New Favorite Web (Micro)Framework
author: Michael
type: post
date: 2011-05-10
url: /2011/05/nina-my-new-favorite-web-microframework/
dsq_thread_id:
  - 527345612
categories:
  - Programming
tags:
  - .net
  - 'c#'
  - html
  - json
  - nina
  - web
---
One of the things I&#8217;m excited to see is the huge increase in Open Source projects in the .NET world. NuGet has certainly helped the recent explosion, but even before that there have been numerous projects gaining legs in the .NET community. Even better, the movement has been learning from other programming ecosystems to bring some great functionality into all kinds of .NET based systems.

One of my favorite projects on the scene is [Nina, a Web Micro Framework][1] by [jondot][2]. What exactly is a web micro framework? Quite simply it easily allows you to go from an HTTP request to a server side method call with little friction. The project is inspired by [Sinatra][3] a very popular ruby framework for server-side interaction which doesn&#8217;t involve all the overhead of a convention based framework like Ruby on Rails.

## Wait, Isn&#8217;t This .MVC?

Sort of- but the two frameworks take very different approaches in how they map an HTTP request to a function call. .MVC is a huge improvement over &#8220;that which must not be named&#8221; but still abstracts the underlying HTTP request/response: controllers and actions to handle logic, models to represent data, views to render results, and routing to figure out what to do. This is usually a good thing as you can easily get fully formed objects into and out of the server in an organized way and has incredible benefits over WebForms. But sometimes that is too much for what you want or need. In our ajax driven world we simply want to do something&#8211;GET or POST some data&#8211;as quickly and easily as possible. We don&#8217;t want to set up a routing for new controller, create a model or view model, invoke an action, return a view, and all that other stuff; we just want to look at the request and do something. That&#8217;s where Nina comes in- it elegantly lets you &#8220;think&#8221; in HTTP by providing an API to do something based on a given HTTP request. It&#8217;s extremely lightweight and extremely fast. It&#8217;s the bare essentials of MVC by providing a minimalist view of functionality in a well defined DSL. On the plus side, the MVC framework and Nina can complement each other quite well (Nina can also stand on its own, too!). Let&#8217;s take a look.

## How It Works

Nina is essentially functionality added to a web project in the same way the MVC bits are added to a web project. It&#8217;s not an entirely new HTTP server implementation. It&#8217;s powered off of the standard .NET HttpApplication class and unlike the various [OWIN][4] toolkits Nina doesn&#8217;t try and rewrite the underlying HttpContext or IIS server stack. To start things off Nina is powered by creating a class that handles all requests to a given url, referred to as an endpoint. This class inherits from _Nina.Application_ and handles all requests to that endpoint- no matter what the rest of the url is. This is done by &#8220;mounting&#8221; the class to an endpoint in your Global.asax file. It&#8217;s not too different than setting up a routing for MVC. However, instead of MVC, you&#8217;re not routing directly to specific actions or a pattern of actions, but &#8220;gobbing&#8221; up all requests to that url endpoint. Below is an example of a global.asax file from the Nina demo project. There are two Nina applications- the Contacts class gets mounted to the contacts endpoint and Posts gets mounted to the blog endpoint.

<pre class="syntax c#">private static void RegisterRoutes()
    {
            RouteTable.Routes.Add(new MountingPoint("contacts"));
            RouteTable.Routes.Add(new MountingPoint("blog"));
    }</pre>

When you&#8217;re mounting an endpoint any request to that endpoint will go to that class- and that class will handle everything else. So anything with a url of /contacts, /contacts/123, /contacts/some/long/path/with/file.html?x=1&y=1 will go to the Contacts class. There&#8217;s no automatic mapping of url parts to action names, or auto filling of parameters. That&#8217;s all handled by the class you specify which inherits from Nina.Application. Routing to individual methods is handled within these classes by leveraging the Nina DSL. I like this approach, as it keeps routing logic tied to specific endpoints rather than requiring you to centrally locate everything or to dictate globally how routing should work via conventions. Of course, there are pros and cons in either case. In very complex systems the Global.asax can get quite large; you can certainly refactor routing logic into helper functions as necessary, but moving routing definitions closer to the logic has its benefits. I&#8217;m also not too big of a fan when it comes to attribute based programming so not having to pepper your action methods with specific filters- whether for a Uri template in the case of WCF or Http Verbs for .MVC- is a big plus.

## Handling Requests

This is where the beauty of Nina comes in. Once we&#8217;ve mounted an application to an endpoint we can handle what to do based on two variables: the HTTP method and the path of the request. This is done via four function calls which are part of the Nina.Application class and map to the four HTTP verbs: Get(), Put(), Post() and Delete(). Each function takes in two parameters: the first is a Uri template which determines when this method gets invoked. The second is a lambda with a signature of Func<NameValueCollection, HttpContext, ResourceResult>. This lambda is what gets invoked when the current requests matches the Uri template. The first parameter are the template parts (explained later), the second parameter is the underlying HttpContext object, and the Function returns a Nina.ResourceResult class. For all intents and purposes a ResourceResult is similar to an ActionResult in .MVC. Nina provides quite a number of ResourceResults, from Html views to various serialization objects to binary data.

This setup is powered by an extremely nice DSL for handling function invocation from HTTP requests and yields a very nice description of your endpoint. You specify the HTTP verb required to invoke the function. You specify the Uri template to when that match should occur&#8211;very similar to setting up routes&#8211;and your handler is actually a parameter, which you can specify inline or elsewhere if needed. The Uri templating is pretty slick, as it allows any level of fuzzy matching. Because the template is automatically parsed and passed as a variable to your handler, you can easily get out elements of the Uri using the template tokens in your Uri. Let&#8217;s take a look at a simple example:

Take a look at the example application below.

<pre class="syntax c#">public class Contacts : Nina.Application
    {
        public Contacts()
        {
            Get("", (m, c) =&gt;
                        {
                            //Returns anything at the root endpoint, i.e. /contacts
                            var data = SomeRepository.GetAll();
                            return Json(data);
                        });
            Get("Detail/{id}", (m, c) =&gt;
                                   {
                                       //Returns /contacts/detail/XYZ

                                       //m is the bound parameters in the template
                                       //this will be a collection with m["ID"] returning XYZ
                                       var id = m["ID"]; //returns XYZ

                                       var data = SomeRepository.GetDetail(id);
                                       return View("viewname", data); //Nina has configurable ViewEngines!
                                   });

            Post("", (m,c) =&gt;
                         {
                             //A post request to the root endpoint.

                             return Nothing(); 
                         });
         }
}</pre>

We&#8217;re exposing three operations: two GET calls and one POST. We&#8217;re handling a GET and POST operation at the endpoint root. In our global.asax we&#8217;ve mounted this application at /contacts, so everything here is relative to /contacts. A template of &#8220;&#8221; will simply match a Uri of /contacts. If we wrote  <code class="syntax c#">RouteTable.Routes.Add(new MountingPoint("contacts"));</code> in our Global.asax than this class would be at the root of our application, i.e. &#8220;http://localhost/&#8221;. Finally, we have another GET call at /detail/{id}. This is actually a URI template, similar to a Route, so anything which matches that template will be handled by that function. In this case /detail/123 or /detail/xyz would match. The template variables are passed as a key/value array in the &#8220;m&#8221; parameter of the lambda and can easily be pulled out. These are your template parts that are automatically parsed out for you.

Using this DSL we can create any number of handlers for any GET, POST, PUT or even DELETE request. We can easily access HTTP Headers, Form variables, or the Request/Response objects from the HttpContext class. Most importantly we can easily view how a request will get handled by our system. The abstraction that MVC brings via Routes, Controllers and Actions is helpful; but not always necessary. Nina provides a different way of describing what you want done that serves a variety of purposes.

## Returning Results

So far we&#8217;ve focused on the Request side of Nina and haven&#8217;t delved too much in the Response side. Nina&#8217;s response system is very similar to .MVC&#8217;s ActionResult infrastructure. Nina has a suite of classes which inherit from ResourceResult that allows you to output a response in a variety of ways. You can serialize an object into Json or Xml, render straight text, return a file, return only a status code, or even return a view. Nina supports numerous View engines&#8211;including Razor but also NHaml, NDjango and Spark&#8211;that&#8217;s beyond the scope of this blog but worth checking out. I&#8217;m a big fan of Haml. Results are returned using one of the method calls provided through the Nina.Application class and should serve all your needs. The best thing to do is explore the Nina.Application class itself and find out which methods return ResourceResults objects.

## This is cool, but why use it?

The great part about Nina is that even though it can stand alone as an application, it can just as easily augment an existing WebForms (Blah!) or MVC application via mounting endpoints using the Routing engine. There are times when you want speed and simplicity for your web app rather than a fully-fledged framework. MVC is great, but requires quite a few moving parts and abstracts away underlying HTTP. The new Restful Web API&#8217;s Microsoft is rolling out for WCF are also nice, but I&#8217;ve never been a fan of attribute based programming and the WCF endpoints are service specific. Nina offers much more flexibility. Nina strikes the right balance by honoring existing HTTP conventions while providing flexibility of output. Sinatra, Nina&#8217;s inspiration, came about by those who didn&#8217;t want to follow the Rails bandwagon and the MVC convention it implemented. They wanted an easier, lightweight way of parsing and handling HTTP requests, and that&#8217;s exactly what Nina does.

Here are some use cases where Nina works well:

  * Json powered services. Even though MVC has JsonResult, Nina provides a low friction way of issuing a get request to return Json data, useful for Autosuggest lists or other Json powered services. JQuery thinks in terms of get/post commands so mapping these directly to mounted endpoints becomes much more fluid. One of my more popular articles is the [New Web App Architecture][5]. Nina provides a nice alternative to Json powered services that can augment one of the newer javascript frameworks like [Knockout][6] or [Backbone][7].
  * Better file delivery. HttpHandlers work well, but exist entirely outside the domain of your app. Powering file delivery through Nina- either because the info is in a data store or required specific authentication, works well.
  * Conventions aren&#8217;t required. Setting up routings, organizing views, and implementing action methods all require work and coding. Most of the time, you just want to render something or save something. Posting a search form, save a record via ajax, polling for alerts are all things that could be done with the conventions of MVC but aren&#8217;t necessarily needed. Try the lightweight approach of Nina and you&#8217;ll be glad you did. With support for View engines you may even want to come up with your own conventions for organizing content.

When the time it takes to do something simple simply becomes too great you&#8217;re using the wrong tool. I strongly encourage you to play around with Nina&#8211; you&#8217;ll soon learn to love the raw power of HTTP and the simplicity of the API. It will augment your existing tool belt quite well and you&#8217;ll find how much you can do when when you can express yourself in different ways.

 [1]: http://jondot.github.com/nina/
 [2]: http://twitter.com/#!/jondot
 [3]: http://www.sinatrarb.com/
 [4]: http://owin.org/
 [5]: http://www.michaelhamrah.com/blog/2010/08/the-new-webapp-architecture-asp-net-mvc-3-jquery-templating-with-pure-and-the-json-value-provider/
 [6]: knockoutjs.com
 [7]: http://documentcloud.github.com/backbone/
