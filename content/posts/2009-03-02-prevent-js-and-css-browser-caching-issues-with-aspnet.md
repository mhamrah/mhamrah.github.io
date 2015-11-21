---
title: Prevent Js and Css Browser Caching Issues with ASP.NET
author: Michael
type: post
date: 2009-03-02
url: /2009/03/prevent-js-and-css-browser-caching-issues-with-aspnet/
dsq_thread_id:
  - 527224601
categories:
  - Programming
tags:
  - aspnetmvc
---
You&#8217;ve seen this problem before- you deploy a new version of your website but the style is off and you&#8217;re getting weird javascript errors. You know the issue: Firefox or IE is caching and old version of the css/js file and it&#8217;s screwing up the web app. The user needs to clear the cache so the latest version is pulled. The solution: versionstamp your include files!

Take a lesson from Rails and create a helper which appends a stamp to your include files (and takes care of the other required markup). It&#8217;s simple- embed the following code in your views:

<pre class="brush: csharp; title: ; notranslate" title="">&lt;%=SiteHelper.JsUrl("global.js") %&gt;
</pre>

will render

<pre class="brush: xml; title: ; notranslate" title="">&lt;script type="text/javascript" src="/content/js/global.js?33433651"&gt;&lt;/script&gt;
</pre>

The browser will invalidate the cache because of the new query string and you&#8217;ll be problem free. Version stamps are better than timestamps because the version will only change if you redeploy your site.

Here&#8217;s the code, which is based on the AppHelper for Rob Conery&#8217;s Storefront MVC:

<pre class="brush: csharp; title: ; notranslate" title="">using System.Reflection;
using System.Web.Mvc;

namespace ViewSample
{
public static class SiteHelper
{
private static readonly string _assemblyRevision = Assembly.GetExecutingAssembly().GetName().Version.Build.ToString() + Assembly.GetExecutingAssembly().GetName().Version.Revision.ToString();

/// &lt;summary&gt;
/// Returns an absolute reference to the Content directory
/// &lt;/summary&gt;
public static string ContentRoot
{
get
{
return "/content";
}
}

/// &lt;summary&gt;
/// Builds a CSS URL with a versionstamp
/// &lt;/summary&gt;
/// &lt;param name="cssFile"&gt;The name of the CSS file&lt;/param&gt;
public static string CssUrl(string cssFile)
{

string result = string.Format("&lt;link rel='Stylesheet' type='text/css' href='{0}/css/{1}?{2}' /&gt;", ContentRoot, cssFile, _assemblyRevision);
return result;
}
/// &lt;summary&gt;
/// Builds a js URL with a versionstamp
/// &lt;/summary&gt;
/// &lt;param name="cssFile"&gt;The name of the CSS file&lt;/param&gt;
public static string JsUrl(string jsPath)
{
return string.Format("&lt;script type='text/javascript' src='{0}/js/{1}?{2}'&gt;&lt;/script&gt;", ContentRoot, jsPath, _assemblyRevision);
}

}
}

</pre>
