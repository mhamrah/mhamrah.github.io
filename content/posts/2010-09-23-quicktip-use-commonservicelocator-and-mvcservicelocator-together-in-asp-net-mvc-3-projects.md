---
title: 'QuickTip: Use CommonServiceLocator and MvcServiceLocator together in ASP.NET MVC 3 Pre-Release Projects'
author: Michael
type: post
date: 2010-09-23
url: /2010/09/quicktip-use-commonservicelocator-and-mvcservicelocator-together-in-asp-net-mvc-3-projects/
dsq_thread_id:
  - 527224524
categories:
  - Programming
tags:
  - .net
  - ASP.NET MVC
  - aspnetmvc
---
**UPDATE: This post is outdated since ASP.NET MVC Beta.  Use the DependencyResolver static class instead.**

The integration of the CommonServiceLocator pattern within ASP.NET MVC is a positive step forward for the .MVC framework. Dependency management via ServiceLocation is a smart way to go, especially for large codebases with complex dependency needs. ServiceLocation keeps constructors clean, and prevents bloat in higher-level classes which don&#8217;t need to know about lower-level dependencies.

However, the fact that .MVC now has its own ServiceLocation infrastructure, via the System.Web.Mvc.IServiceLocator interface, is a little troublesome for code which already uses the Microsoft CommonServiceLocator class found in the Unity Enterprise Application Block. But don&#8217;t fret- luckily, the IServiceLocator interface is exactly the same in the System.Web.Mvc namespace and the Microsoft.Practices.ServiceLocation namespace. This means you can have one class implement both interface simultaneously, like so:

<pre class="brush: csharp; title: ; notranslate" title="">public class  SomeServiceLocatorWrapper : System.Web.Mvc.IServiceLocator, Microsoft.Practices.ServiceLocation.IServiceLocator
{
 //Implicity Implementation of Methods
}
</pre>

What&#8217;s even easier is when there&#8217;s already a wrapper class around the IServiceLocator for you, such as the one provided by Unity via the UnityServiceLocator class in Microsoft.Practices.Unity&#8217;s namespace. The following code below provides all the functionality you need to use both ServiceLocators:

<pre class="brush: csharp; title: ; notranslate" title="">public class UnityMvcServiceLocator : UnityServiceLocator, System.Web.Mvc.IServiceLocator
{
 public UnityMvcServiceLocator(IUnityContainer container)
 : base(container)
 {

 }

}
</pre>

Once you have that class in place, then it&#8217;s just a matter of hooking both up in your Global.asax file like so:

<pre class="brush: csharp; title: ; notranslate" title="">//In Global.asax's Application_Start hook:
var container = UnityContainerBuilder.CreateContainer();
var locator = new UnityMvcServiceLocator(container);

ServiceLocator.SetLocatorProvider(() =&gt; locator);
MvcServiceLocator.SetCurrent(locator);
</pre>

This allows you to access the same locator either via the MvcServiceLocator.Current instance or the ServiceLocator.Current instance.
