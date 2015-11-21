---
title: 'Digging into Ruby on Rails from C# and .MVC (Asp.Net MVC)'
author: Michael
type: post
date: 2009-02-18
url: /2009/02/digging-into-ruby-on-rails-from-c-and-mvc-aspnet-mvc/
dsq_thread_id:
  - 527224493
categories:
  - Programming
tags:
  - aspnetmvc
  - rails
---
One of the things I&#8217;ve been doing lately is digging into Ruby on Rails. I&#8217;ve always wanted to learn Rails since I was first exposed to Rails at an AjaxWorld conference in &#8217;06 (at least, I think in 06). David Heinemeier Hansson actually presented!

Alas, I never could devote enough time to get past the tipping point. I&#8217;m comfortable with C# and ASP.NET and evolving those skills has always been the priority. But the perfect storm has happened recently- in order to save space in my apartment, I got rid of my PC desktop and now only use my MacBook. I got tired of using Parallels and Visual Studio, and a new project came up in which I could either use ASP.NET MVC (which I&#8217;m calling .MVC from now on) or RoR. I thought it was time to finally try RoR.

The verdict is Rails is great. I still can&#8217;t &#8220;express&#8221; myself as well as I want to with Rails, but in comparison to .MVC Rails is pretty slick, and the evolution of Rails (specifically, [combining Merb with Rails][1]) is pretty exciting.  Most importantly, learning Rails has actually made me a better C#/ASP.NET MVC developer- if you&#8217;re working with .MVC you have to spend at least an afternoon playing around with Rails- you&#8217;ll get an invaluable perspective on MVC and web programming.

Now, a discussion on any programming language/framework always causes a heated debated.  I&#8217;m not an expert (or even a beginner) on Ruby or Rails, but these are my initial impressions.

**Ruby, as a Language
  
** 

One major leap between .MVC and Rails is Ruby as a language. Yes, Ruby and C# are both OO languages, but Ruby is a [dynamic language][2]&#8211; and if you&#8217;re up on C# 4.0, you&#8217;ll know that C# is on it&#8217;s way to [becoming a dynamic language too][3]. So if you want keep your C# skills on the cutting edge, get a head start on a full fledged dynamic language!

I originally made the mistake of jumping in and assuming Ruby was more vb- or js- esque than it actually was.  It&#8217;s pretty smart in the way it behaves, almost a tailored version of C#.  Ruby&#8217;s use of [symbols][4] is pretty big difference over other languages that&#8217;s extensively used in Rails and pretty handy.

Lambda expressions are also core part of Ruby, and are becoming a more integral part of C#.  This allows Ruby to be extremely concise in getting things done- which is awesome when you know what you&#8217;re doing.  When you don&#8217;t it can be confusing.  But lambdas make sense and are awesome when you know how to use them- and knowing Ruby can help you grasp lambda expressions in C#.

Rails leverages Ruby&#8217;s dynamic language to make a lot happen under the hood.  I personally found letting the Rails framework do its thing as one of the biggest hurdles in learning Rails.  I wanted to either program or explicitly orchestrate everything!  One prime example is the persistence model in Rails-  I struggled to figure out how properties where set in models from migration classes- but it&#8217;s entirely automatic!  Also, a lot of methods are created dynamically.  This allows an extremely fluid expression in writing code, making Ruby a pretty natural programming language.  (Example: the find\_by ActiveRecord methods; declaring links like edit\_xxx_path).

C# is making its way into being a more fluid programming language similar to Ruby.  Meaning, writing code and describing code are converging to be the same thing.  This is seen in lambda expressions and fluent interfaces, where you can daisy chain methods together.  [Moq][5] is a good example of the fluent interface approach.  I wouldn&#8217;t be surprised if there&#8217;s even a larger convergence with C# and Ruby in the years to come.

**Rails, as a Framework**

Those in the .NET world who follow [Jeremy Miller][6] have probably heard the term of [opinionated software.][7] He talks about opinionated software in the context of [FubuMVC][8], an alternative to .MVC written in an opinionated style.  It&#8217;s worth checking out- it&#8217;s very interesting to use but Rails still cracks the MVC shell wide open because it leverages Ruby&#8217;s language features.

Opinionated software originally came from the Adam and Eve of Rails, [37Signals][9].  Rails is highly opinionated- which is a blessing and curse.  The great thing about Rails is that it does what it does extremely well.  The MVC triangle are completely separate but seamlessly integrated, and extension points are explicit.  It&#8217;s how MVC should be- and .MVC doesn&#8217;t come close to Rails as an MVC framework.  Hate NHibernate configuration?  You&#8217;ve never seen an ORM framework until you&#8217;ve used ActiveRecord.  Really want to abandon code-behind files and excessive code in views? Really want to do TDD (and even BDD)?  Rails right now is the iPhone in a tin cup and string world.

Here are some highlights:

  1. Respond_to method for rendering html, xml, json or whatever from a single controller.  .MVC ActionResult could learn a thing or two.
  2. Rails&#8217; partial layouts, partial actions and partial views are pretty slick compared .MVC&#8217;s master pages, partial views, and partial layouts.  A prime example is using Rails partial views to render a collection of objects.
  3. Passing data between controllers and actions is a lot slicker than .MVC ViewData
  4. Rails helpers are a lot more helpful.
  5. Rails is pretty slick when it comes to mapping between posted data and Models- a lot better than binders.
  6. And of course, Rails&#8217; database integration will make you wonder why you ever spent more than 5 minutes learning about databases- it&#8217;s such a model centric approach with migrations you&#8217;ll hate doing any data tier work in .MVC.  (Although learning databases are extremely important no matter what language/framework you use).
  7. Plugins.  Rails Plugins are simply awesome- extremely fluid integration into your Rails application.  And combining plugins and other support with gem makes maintenance and upgrades a breeze (forget the GAC!)

**Where .MVC Shines**

I&#8217;ve been hyping up Rails a lot, but there is a major disadvantage with Rails: good luck straying from the Rails track.  If you get off the train, you&#8217;re walking alone.  This is something .MVC does well- allows you to change the game with whatever architecture pattern you wish.  It&#8217;s one reason I love .NET- you can do literally do whatever want (yes, you can whatever you want with any language, but the .NET framework is pretty awesome).  .MVC is extremely extensible- from replaceable view engines, routing engines, controller factories, coupled with whatever architecture pattern you want.  And all very testable.  C#&#8217;s usage of extension methods also add pretty nice extensibility to classes, too.

In fact, Rails&#8217; plans to integrate with Merb as way to be more modular is a prime example of something .MVC already does well- allows you to mix and match what you want instead of forcing conformance.  Allowing total and explicit control over the application stack- including your architecture- is a great advantage in maintaining the health of ongoing software. Yes, you can evolve Rails to do what you want, but the transparency .NET offers is something I don&#8217;t see in Rails just yet.  It&#8217;s definitely a pro and a con- on the one hand, Rails offers fluid interaction between components.  On the other, .NET doesn&#8217;t force you into any specific pattern.

**In Conclusion**

The bottom line is, check out Rails.  The best way to get started is with [the Rails guides][10].  From a .MVC perspective, you&#8217;ll learn a lot about MVC and what an MVC framework can do- and it will help you out in your .NET development.  Knowing Ruby will also keep your C# skills sharp and help you in becoming a well rounded developer.  I&#8217;ll even bet you carry over some Rails tricks to your next .MVC app!

[<img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f02%2fdigging-into-ruby-on-rails-from-c-and-mvc-aspnet-mvc%2f&#038;bgcolor=000099" border="0" alt="kick it on DotNetKicks.com" />][11]

 [1]: http://rubyonrails.org/merb
 [2]: http://en.wikipedia.org/wiki/Dynamic_programming_language
 [3]: http://ironpython-urls.blogspot.com/2008/12/c-becomes-dynamic-language.html
 [4]: http://glu.ttono.us/articles/2005/08/19/understanding-ruby-symbols
 [5]: http://code.google.com/p/moq/
 [6]: http://codebetter.com/blogs/jeremy.miller/
 [7]: http://codebetter.com/blogs/jeremy.miller/archive/2008/10/23/our-opinions-on-the-asp-net-mvc-introducing-the-thunderdome-principle.aspx
 [8]: http://code.google.com/p/fubumvc/
 [9]: http://gettingreal.37signals.com/ch04_Make_Opinionated_Software.php
 [10]: http://guides.rails.info/
 [11]: http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f02%2fdigging-into-ruby-on-rails-from-c-and-mvc-aspnet-mvc%2f
