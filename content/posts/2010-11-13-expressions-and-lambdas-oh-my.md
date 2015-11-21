---
title: 'Expressions and Lambdas: Oh My!'
author: Michael
type: post
date: 2010-11-13
url: /2010/11/expressions-and-lambdas-oh-my/
dsq_thread_id:
  - 527224522
categories:
  - Programming
tags:
  - .net
  - ASP.NET MVC
  - aspnetmvc
---
I&#8217;ve been working on a toolkit called [Redaculous][1]&#8211; it&#8217;s a .NET Library for the really cool key/value store [Redis][2].  It&#8217;s built on top of the [ServiceStack.Redis][3] library, which provides various .NET clients for Redis.  Redaculous is meant to make aggregating Redis commands a little easier- but don&#8217;t get too excited.  The project __is in its infancy, and will undergo many changes, if it even gets off the ground.  This post isn&#8217;t about Redis nor Redaculous- it&#8217;s about how parts of Redaculous leverage [Expressions][4] and [Lambdas][5] to drive a lot of the functionality Redaculous is meant to provide, and how you can leverage Expressions to make your programming life easier.  ASP.NET MVC and lots of other great frameworks do it, so why can&#8217;t you?

The problem was simple:  I have a class, with a bunch of properties, and those properties have values.  In order to put them into Redis, I need to know the property name and the value it contains.  I need to know, at runtime, that myObj.SomeProperty has a property called &#8220;SomeProperty&#8221; and the value of that property.  This is a problem shared with most serialization tools and ORM mappers:  how does a &#8220;SomeProperty&#8221; make it to a column in table in a database, or make it to a node in xml?

This problem can (and has been) solved in a variety of ways.  The most common was attributes to decorate classes and properties- which is how WCF constructs contracts or how some ORM toolkits work.  But that&#8217;s extremely intrusive, in this pseudo example which combines WCF and DataAccess :

<pre class="syntax c# escaped">[Table("DtoTb")]
[DataContract]
public class Dto
{
     [PrimaryKey]
     [DataMember]
     public int Id { get; set; }

     [ColumnName("Name")]
     [DataMember]
     public string Name { get; set; }
}
</pre>

There&#8217;s a weird combination of data storage and data definition which goes on in toolkits which use that approach.  It&#8217;s not really a smooth way to operate- and the approach falls out when the model greatly diverages from the underlying table, or when a class has numerous other complex types.  Worse, when we want to also expose that class via WCF, we&#8217;re essentially tacking on both data access description and web service schema description.  How many attributes can you tack on there? Validation attributes, too?

Another approach is to use code-generation.  This is essentially what Linq-to-Sql does; it provides the underlying property descriptions at design time, saving a lot of manual typing.  I&#8217;m not a big fan of code generators; they have their place, but you lose a lot of control in defining explicit functionality.  You&#8217;re boxed into what is generated and most customizations usually don&#8217;t fit in well: when you stray outside of what the code generator provides (or even outside of what the core generation is focused on) you find yourself trying to shove a square peg in a round hole.

The best approach is to do what great tools like [Norm][6], [Fluent NHibernate,][7] [AutoMapper][8], and .MVC&#8217;s Html helpers do: they use [Expressions][5].  Most often the configuration is offloaded to separate classes which rely heavily on Expressions for property introspection.  This allows an end user to write code like:

<pre class="syntax c# escaped">//For MVC:
Html.TextBoxFor(m =&gt; m.ProductName);
//Or for Fluent NHibernate:
Id(x =&gt; x.ImageId);
</pre>

The TextBoxFor creates an input html element with a name=&#8221;ProductName&#8221; attribute; the Id() says NHIbernate should use a class&#8217;s ImageId Property as the Id for the table.

Expressions where introduced in .NET 3.5, and have been heavily leveraged ever since to provide a level of meta-programming which previously didn&#8217;t exist in the .NET world.   Expressions greatly differ from Reflection by the fact Expressions are code which has been parsed into a set of various descriptive classes, while Reflection is compiled code which has been deconstructed into a descriptive semantic. An Expression statement will not actually &#8220;do&#8221; anything.  It can, however, be compiled into a callable function- which is the core advantage over Reflection.  With Expressions, you can have both the description of code and the actual, runnable code together.  Most frameworks create a package or container around the two as a performance optimization- it prevents an application from having to compile the expression multiple times.  The performance is much greater than using reflection to dynamically invoke or inspect properties.

**How It Works**

Expressions and Lambdas go hand-in-hand, and can often be confused with one another. Let&#8217;s take a look at the following code:

<pre class="syntax c# escaped">Expression&lt;Func&lt;SomeDto, string&gt;&gt; expressionLambda = m =&gt; m.SomeStringProp;
Func&lt;SomeDto, string&gt;&gt; funcLambda = m =&gt; m.SomeStringProp;
</pre>

What&#8217;s the difference between the two?  It&#8217;s the same value, right?  Well, running that snippet in Visual Studio, and viewing the debugger properties, you&#8217;ll find two different results for the m=>m.SomeStringProp statement. m=>m.SomeStringProp is the lambda: it&#8217;s simply an alternative way of writing C# for various purposes. Usually, Lambdas are either Func or Action statements used as an alternative for delegate methods. The compiler will generate runnable IL code for the m => m.SomeStringProp statement and create a callable method expecting an instance of SomeDto as the input. In the above example, you could get the value of a SomeStringProp using funcLambda like so:

<pre class="syntax c# escaped">var dto = new SomeDto() { SomeStringProp = "Hell Yeah!" };
funcLambda(dto); //Returns "Hell Yeah!"
</pre>

This can provide a level of agnosticism by passing around logic as variables in a much easier way than vanilla delegates.

Expressions, on the other hand, don&#8217;t compile into runnable IL code.  You can&#8217;t write _funcExpression(dto)_ in the same was a normal lambda.  The compiler does something different: it actually parses the m=>m.SomeStringProp into various expression components which can be traversed, manipulated, rearranged and even compiled into a callable action, as if it were a Func all along. The [I&#8217;ve been working on a toolkit called [Redaculous][1]&#8211; it&#8217;s a .NET Library for the really cool key/value store [Redis][2].  It&#8217;s built on top of the [ServiceStack.Redis][3] library, which provides various .NET clients for Redis.  Redaculous is meant to make aggregating Redis commands a little easier- but don&#8217;t get too excited.  The project __is in its infancy, and will undergo many changes, if it even gets off the ground.  This post isn&#8217;t about Redis nor Redaculous- it&#8217;s about how parts of Redaculous leverage [Expressions][4] and [Lambdas][5] to drive a lot of the functionality Redaculous is meant to provide, and how you can leverage Expressions to make your programming life easier.  ASP.NET MVC and lots of other great frameworks do it, so why can&#8217;t you?

The problem was simple:  I have a class, with a bunch of properties, and those properties have values.  In order to put them into Redis, I need to know the property name and the value it contains.  I need to know, at runtime, that myObj.SomeProperty has a property called &#8220;SomeProperty&#8221; and the value of that property.  This is a problem shared with most serialization tools and ORM mappers:  how does a &#8220;SomeProperty&#8221; make it to a column in table in a database, or make it to a node in xml?

This problem can (and has been) solved in a variety of ways.  The most common was attributes to decorate classes and properties- which is how WCF constructs contracts or how some ORM toolkits work.  But that&#8217;s extremely intrusive, in this pseudo example which combines WCF and DataAccess :

<pre class="syntax c# escaped">[Table("DtoTb")]
[DataContract]
public class Dto
{
     [PrimaryKey]
     [DataMember]
     public int Id { get; set; }

     [ColumnName("Name")]
     [DataMember]
     public string Name { get; set; }
}
</pre>

There&#8217;s a weird combination of data storage and data definition which goes on in toolkits which use that approach.  It&#8217;s not really a smooth way to operate- and the approach falls out when the model greatly diverages from the underlying table, or when a class has numerous other complex types.  Worse, when we want to also expose that class via WCF, we&#8217;re essentially tacking on both data access description and web service schema description.  How many attributes can you tack on there? Validation attributes, too?

Another approach is to use code-generation.  This is essentially what Linq-to-Sql does; it provides the underlying property descriptions at design time, saving a lot of manual typing.  I&#8217;m not a big fan of code generators; they have their place, but you lose a lot of control in defining explicit functionality.  You&#8217;re boxed into what is generated and most customizations usually don&#8217;t fit in well: when you stray outside of what the code generator provides (or even outside of what the core generation is focused on) you find yourself trying to shove a square peg in a round hole.

The best approach is to do what great tools like [Norm][6], [Fluent NHibernate,][7] [AutoMapper][8], and .MVC&#8217;s Html helpers do: they use [Expressions][5].  Most often the configuration is offloaded to separate classes which rely heavily on Expressions for property introspection.  This allows an end user to write code like:

<pre class="syntax c# escaped">//For MVC:
Html.TextBoxFor(m =&gt; m.ProductName);
//Or for Fluent NHibernate:
Id(x =&gt; x.ImageId);
</pre>

The TextBoxFor creates an input html element with a name=&#8221;ProductName&#8221; attribute; the Id() says NHIbernate should use a class&#8217;s ImageId Property as the Id for the table.

Expressions where introduced in .NET 3.5, and have been heavily leveraged ever since to provide a level of meta-programming which previously didn&#8217;t exist in the .NET world.   Expressions greatly differ from Reflection by the fact Expressions are code which has been parsed into a set of various descriptive classes, while Reflection is compiled code which has been deconstructed into a descriptive semantic. An Expression statement will not actually &#8220;do&#8221; anything.  It can, however, be compiled into a callable function- which is the core advantage over Reflection.  With Expressions, you can have both the description of code and the actual, runnable code together.  Most frameworks create a package or container around the two as a performance optimization- it prevents an application from having to compile the expression multiple times.  The performance is much greater than using reflection to dynamically invoke or inspect properties.

**How It Works**

Expressions and Lambdas go hand-in-hand, and can often be confused with one another. Let&#8217;s take a look at the following code:

<pre class="syntax c# escaped">Expression&lt;Func&lt;SomeDto, string&gt;&gt; expressionLambda = m =&gt; m.SomeStringProp;
Func&lt;SomeDto, string&gt;&gt; funcLambda = m =&gt; m.SomeStringProp;
</pre>

What&#8217;s the difference between the two?  It&#8217;s the same value, right?  Well, running that snippet in Visual Studio, and viewing the debugger properties, you&#8217;ll find two different results for the m=>m.SomeStringProp statement. m=>m.SomeStringProp is the lambda: it&#8217;s simply an alternative way of writing C# for various purposes. Usually, Lambdas are either Func or Action statements used as an alternative for delegate methods. The compiler will generate runnable IL code for the m => m.SomeStringProp statement and create a callable method expecting an instance of SomeDto as the input. In the above example, you could get the value of a SomeStringProp using funcLambda like so:

<pre class="syntax c# escaped">var dto = new SomeDto() { SomeStringProp = "Hell Yeah!" };
funcLambda(dto); //Returns "Hell Yeah!"
</pre>

This can provide a level of agnosticism by passing around logic as variables in a much easier way than vanilla delegates.

Expressions, on the other hand, don&#8217;t compile into runnable IL code.  You can&#8217;t write _funcExpression(dto)_ in the same was a normal lambda.  The compiler does something different: it actually parses the m=>m.SomeStringProp into various expression components which can be traversed, manipulated, rearranged and even compiled into a callable action, as if it were a Func all along. The][4] has all of the available types a lambda expression can be broken down into.  Essentially, a hierarchal tree is generated, with each distinct component of the lambda expression being represented by one of the many Expression classes.  These <span style="font-size: 13.2px;">classes can be used to inspect various aspects of that part.  The above lambda is simply a <a href="http://msdn.microsoft.com/en-us/library/system.linq.expressions.memberexpression.aspx">MemberExpression</a>, which is used for field and properties. Using MemberExpressions you can get the name of the Member, the Property Type, you can use the NodeType property- that&#8217;s available with all Expression classes- to learn that it&#8217;s a MemberAccess call.</span>

<span style="font-size: 13.2px;">Because Expressions can be compiled into runnable code, frameworks get the best of both worlds: metadata about the call, and the call itself.  This is the precisely how ASP.NET MVC Html Helpers are built: that TextBoxFor method takes in an expression which it uses to generate the Html output. The Html helpers in MVC inspect the input expression to figure out what the name of the property for the html name attribute value, and then it runs the compiled expression against the current ViewModel object to get the value for the value html attribute.  The expression metadata and compiled function is actual cached in various static classes for performance: you don&#8217;t want to have to compile the expression every time you use it.  The performance is much better than using reflection alone.</span>

Redaculous is using Expressions to avoid the magic string conundrum- by using expressions, Redaculous can parse a statement like &#8220;m = m.Name&#8221; and know it should put the value of a class&#8217;s Name property in the store using a key involving the word &#8220;Name&#8221; in some way. You should explore the [MVC Framework&#8217;s source code][9] to dig in to how they&#8217;re using expressions for strongly typed helpers. Both Norm and Automapper have some pretty straightforward usage too. By inspecting how other tools use these features you can more easily integrate them into your own projects- and increase productivity by eliminating redundant code and code smells involving magic strings.

Other languages, like Ruby, have a similar level of functionality built in. This is inherit in all dynamic languages: functionality to not only provide an operation, but functionality to describe that operation as well.  I still get some hits on my [ASP.NET MVC and Rails][10] article, and I&#8217;d say one of the biggest differences is how Rails, and Ruby in general, use dynamic language features like code metadata to drive functionality.  .NET has always had reflection, but expressions provide a much easier, and much more performant way of dealing with meta-programming.  <span style="font-size: 13.2px;">Expressions provide a core of the functionality in the Dynamic Language Runtime and have always been a strong part of Linq&#8217;s roots.  DLR capabilities, using expressions, will continue to increase its surface area in the .NET world and should be a part of any developers toolkit.</span>

 [1]: https://github.com/mhamrah/Redaculous
 [2]: http://code.google.com/p/redis/
 [3]: http://code.google.com/p/servicestack/wiki/ServiceStackRedis
 [4]: http://msdn.microsoft.com/en-us/library/bb506649.aspx
 [5]: http://msdn.microsoft.com/en-us/library/bb397687.aspx
 [6]: https://github.com/atheken/NoRM
 [7]: http://fluentnhibernate.org/
 [8]: http://automapper.codeplex.com/
 [9]: http://aspnet.codeplex.com/wikipage?title=MVC&referringTitle=Home
 [10]: http://www.michaelhamrah.com/blog/2009/02/digging-into-ruby-on-rails-from-c-and-mvc-aspnet-mvc/
