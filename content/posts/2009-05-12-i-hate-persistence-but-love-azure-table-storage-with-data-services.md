---
title: I Hate Persistence (but love Azure Table Storage)
author: Michael
type: post
date: 2009-05-12
url: /2009/05/i-hate-persistence-but-love-azure-table-storage-with-data-services/
syntaxhighlighter_encoded:
  - 1
dsq_thread_id:
  - 527224580
categories:
  - Programming
tags:
  - azure
  - dotnet
  - orm
---
If there&#8217;s one thing I can&#8217;t stand in the development process it&#8217;s writing code to save data.  In fact, there are only a few things which I&#8217;d consider more useless than dealing with data persistence- one of them being data migration.  I hate dealing with persistence because it&#8217;s totally mundane and repetitive code.  Worse, nobody outside of IT really understands the details of persistence.  Which means it has no business value.  And why should users care about unit of work or active record vs. repository?   Should saving an object really be that complicated of a task? Absolutely not.  Which is why no one cares.  It&#8217;s like caring about how your beer was delivered.  Really, I just want to drink the damn beer.  Yet somehow the thing which should be a no brainer takes the most time and effort to do.  And causes the most debate.  And causes the most problems.  However, I recently got my first glimmer of hope- a _cloud_ with a silver lining- that maybe, just maybe, someday, we&#8217;ll actually have simple data access.

We as a development community have been transfixed on data access strategies.  It&#8217;s the great elephant in the room, and has totally consumed all our attention.  It&#8217;s distracting.  Only a .NET/Java language war in &#8217;04 could rival such a fierce debate as between [Ayende][1] and [Greg Young][2] over the repository pattern. (My two cents: ActiveRecord. Just kidding, ActiveRecord sucks (not SRP). Use [Subsonic][3]&#8211; which lost me at &#8220;drag and drop&#8221;, but [I would not want to piss off Rob Conery][4]).

Why do I hate persistence?  It&#8217;s never really gotten any better.  The first major advancement came with sprocs.  Everything had to be a sproc, and if you weren&#8217;t using sprocs then you just weren&#8217;t cool.  You can do these great things with sprocs- like writing select statements with inner joins.  But not in your vbscript!  This mostly lead to awesome interview questions like &#8220;why are sprocs important&#8221; and dba&#8217;s who would whine about developers writing bad sql (guess what- they can write bad code too)!

Next, and silently from the Java community, we were given NHibernate and the concept of an object relational mapper.  Suddenly, an awakening occurred, and we as developers could now stick it to those dba&#8217;s and there precious sprocs.  Try changing my auto generated sql, you bastards! Unfortunately, the NHibernate craze did not take off.  We gave up sprocs for those awful configuration files- as if things could get any worse!  I personally have a big chip on my shoulder because NHibernate comes from the Java community.  Java, after all, just sucks (sidenote: this a totally baseless claim).  Fluent NHibernate doesn&#8217;t make it that much better: I&#8217;m not digging Cascade. All(). Inverse(). WithForeignKey(). WithIndex(). WithConvention(). ButNotForThisClass(). How. Many. Can. I. Add() and double code associations for parent/child relationships etc.  Not a big win.  But at least we can rub Fluent NHibernate in Hibernate&#8217;s xml hell!  Yes, there&#8217;s the auto mapper configuration which I admit is slick- but you&#8217;re kind of locked into the default convention (over configuration).  Stray from the path and you&#8217;re back in the configuration quicksand.  And don&#8217;t even get me started on the NHibernate query syntax.  Yes, there&#8217;s Linq to NHibernate, but why use that when you go directly to the best thing to come out of Microsoft since [the Steve Ballmer video][5].

Finally there was a breath of fresh air, and Microsoft was awesome again.  We were given Linq to Sql.  But when Microsoft realized they did something cool, they quickly pushed for Linq to Entities (oh wait- Entity Framework- Linq to Entities sounds like the cool Linq to Sql).  We were quickly back to the WTF relationship we have with Microsoft.  **If you want to know why EF sucks, just try using it.** Or check out [this good series of articles about l2s vs. l2e.][6] I gave up on Entity Framework when I tried setting up an auto generated db value on an insert.  It&#8217;s a pain in the ass and not nearly as simple as Linq To Sql&#8217;s &#8220;This is an auto generated property&#8221; property.

What are we left with?  Just the endless data access debate.  Which in a way is good for us because **it keeps us from actually delivering something our users care about- like an app that saves data**.  (Um, why can&#8217;t I change the customer address and overcharge them for adventure works gear at the same time??!?!?!)

So where&#8217;s the glimmer of hope?  Azure Table Storage- which might put us back in the Microsoft is cool again category.  Don&#8217;t worry Microsoft- you only support sorting on only one field (&#8220;Choose&#8230; Choose Wisely&#8221;), so I&#8217;m sure you can still spark lots of criticism.  But please give large bonuses to the people behind ATS.  Why is it so great?  The same reason Linq to Sql is great- it just works. ATS with ADO.NET Data Services is about as easy as you can do persistence without actually having to do it. (Side note: I&#8217;m ignoring object databases).  But you really have true POCO storage.  No messy attributes, no base classes, no schema generators, no xml configuration files, no .edmx files.  You have an object- it has public getters and setters- and you can save it.  That&#8217;s all you have to do.  The catch?  A row key and a partition key (Sidenote: I&#8217;m ignoring the whole deploy/run cloud part).  I could live without the partition key, but you&#8217;re not really asking for much.  What else? It&#8217;s built on Linq- and Linq, as we all know, is awesome (just like sql, but no strings!). (Sidenote: more criticism fuel: not all Linq operators are supported).

What else?  ATS is also schema free, so no more entity/table mappings.  Just save an object.  In fact, save different objects in the same table.  Like parent/child classes.  Together.  Like an aggregate root.  Then search on any field (sorry, still no sorting!)  That&#8217;s all you have to do.  And did I mention there&#8217;s no schema migration?  Because there&#8217;s no schema!  We&#8217;re free! Freedom! Sure, it&#8217;s [not the first schema free storage solution][7]&#8211; but it&#8217;s the only one backed with ADO.NET Data Services.  And that&#8217;s the special sauce which makes it yummy!

Yes, ATS isn&#8217;t perfect.  In fact it&#8217;s far from perfect.  But it&#8217;s a great step in the right direction.  NHibernate, Linq to Sql, EF, sprocs are all usable tools (sorry, didn&#8217;t mean to include EF).  There are some great 3rd party toolkits too, but if I have to pay for data access I&#8217;d like you to just do it or me.  There has been a great effort in the OS community (FluentNHibernate is a huge win) to make this stuff easier, but the bottom line is you&#8217;re doing double programming: you got your db and your code, and all that stuff that sits between is just crap.  Even with FluentNHibernate you&#8217;re just shifting your db programming to configuration.  At the end of the day doing data access well isn&#8217;t even a win because the user still doesn&#8217;t have a damn form!  ATS is different- it&#8217;s just really slick.  It&#8217;s scary slick.  Yes, it&#8217;s not ready for prime time, but it&#8217;s a huge step in the right direction.

 [1]: http://ayende.com/Blog/archive/2009/04/26/the-repositoryrsquos-daughter.aspx
 [2]: http://codebetter.com/blogs/gregyoung/archive/2009/04/24/more-on-repository.aspx
 [3]: http:/www.subsonicproject.com
 [4]: http://blog.wekeroad.com/subsonic/subsonic-to-acquire-nhibernate/
 [5]: http://www.youtube.com/watch?v=8To-6VIJZRE
 [6]: http://naspinski.net/post/Linq-to-SQL-vs-Linq-to-Entities--Revisited.aspx
 [7]: http://aws.amazon.com/simpledb/
