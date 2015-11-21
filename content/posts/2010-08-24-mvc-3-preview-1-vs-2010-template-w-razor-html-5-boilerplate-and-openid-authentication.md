---
title: MVC 3 RC VS 2010 Template w/ Razor, Html 5 Boilerplate and OpenId Authentication
author: Michael
type: post
date: 2010-08-25
url: /2010/08/mvc-3-preview-1-vs-2010-template-w-razor-html-5-boilerplate-and-openid-authentication/
dsq_thread_id:
  - 527224532
categories:
  - Programming
---
\### Updated 3/21/2011:

Now with twitter and oauth support. See \[the latest update\](http://www.michaelhamrah.com/blog/2011/03/updated-mvc3-html5-boilerplate-template-now-with-twitter-and-facebook/)

You can pull the source files from the [GitHub page][1], or [download the zip containing the template][2].

I&#8217;ve created an (arguably) bare-minimum Visual Studio 2010 Template for getting started with MVC 3 Preview 1 using the Razor view engine and some other web goodies. I got tired of the vanilla &#8220;Welcome to MVC&#8221; homepage and I&#8217;m not a fan of the MembershipProvider abstraction and all the Account junk included by default.  This template is meant to provide a bare-bones setup of Html 5 and provide OpenId authentication for users (but not full-on user management)- it&#8217;s a simple cocktail of Html5 Boilerplate and DotNetOpenAuth.  The rest is left up to you to build up!

### <span style="font-weight: normal;">Paul Irish&#8217;s Html5 Boilerplate</span>

Paul Irish created a nice starting point for Html5 websites with his [Html5 Boilerplate][3].  There&#8217;s a lot of goodies in it which go beyond just changing the DOCTYPE.  [Check out the  NetTuts+ official guide to Html5 Boilerplate explained by Paul himself][4].  I&#8217;ve ported over the .9 version with comments so you can see what&#8217;s what.  The only changes made where to replace the url&#8217;s in script and link tags with @Url.Content methods.  The original index page is at the root (you&#8217;ll want to delete, but it&#8217;s a handy reference) and the _Layout.cshtml now has a Header and Script section to use in child pages.  This functionality is used in the LogOn view to set up OpenId authentication.

### <span style="font-weight: normal;">OpenId with DotNetOpenAuth</span>

I was never a fan of the stock MembershipProvider infrastructure and setup of the sql tables required for user management.  Usually, user management requires very specific needs site to site, and the default functionality gets butchered anyway.  So all that stuff gets pulled and replaced with the ease and simplicity of OpenId.  The template uses the [OpenId Selector][5] on the login page and [DotNetOpenAuth][6] with Forms Authentication on the backend.  There&#8217;s no Register page, as the specifics of the required user schema varies too much.  But you should be able to combine the OpenId logic on the Login page with a new Register page to suit your needs.  There&#8217;s also no persistence store integrated into the template as that is left entirely up to you.

### <span style="font-weight: normal;">Future Plans</span>

<span style="font-weight: normal;">I really have no set goals for this template, so it will evolve in a free form manner.  The <a href="http://github.com/mhamrah/Html5OpenIdTemplate">code is on GitHub</a> so feel free to branch and edit as necessary.  I&#8217;m going to add some unit tests around the OpenId logic, and possibly evolve the Account creation in some way.  So that&#8217;s about it!</span>

### <span style="font-weight: normal;">Downloads</span>

<span style="font-weight: normal;">You can pull the source files from the <a href="http://github.com/mhamrah/Html5OpenIdTemplate">GitHub page</a>, or <a href="http://www.michaelhamrah.com/blog/wp-content/uploads/2010/11/Html5OpenIdTemplate.zip">download the zip containing the template</a>.</span>

 [1]: http://github.com/mhamrah/Html5OpenIdTemplate
 [2]: https://github.com/downloads/mhamrah/Html5OpenIdTemplate/Html5OpenIdTemplate.zip
 [3]: http://html5boilerplate.com/
 [4]: http://net.tutsplus.com/tutorials/html-css-techniques/the-official-guide-to-html5-boilerplate/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed:+nettuts+(NETTUTS)
 [5]: http://code.google.com/p/openid-selector/
 [6]: http://www.dotnetopenauth.net/
