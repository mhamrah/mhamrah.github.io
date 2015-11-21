---
title: Performance Tracing For Your Applications via Enterprise Library
author: Michael
type: post
date: 2010-02-13
url: /2010/02/performance-tracing-for-your-applications-via-enterprise-library/
dsq_thread_id:
  - 527224549
categories:
  - Programming
---
Performance is one of the most important, yet often overlooked, aspects of programming.  It&#8217;s usually not something you worry about until it gets really bad.  And at that point, you have layers of code you need to sift through to figure out where you can remove bottlenecks.  It&#8217;s also tricky, especially on complex, process orientated systems, to aggregate performance information for analysis.  That&#8217;s where this [gist for a utility performance tracer class comes into play][1].  It&#8217;s meant to hook into Enterprise Library Logging Block by aggregating the elapsed time or information statements and pass them to the Event Log (or other listeners you&#8217;ve hooked up).  Even though the gist is designed for the Enterprise Library, it can easily be modified for other utilities like Log4Net.  Below is an example of how the entry looks in the EventLog.

<p style="text-align: center;">
  <a href="http://i1.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2010/02/event-log-examoke.png"><img class="size-full wp-image-350 aligncenter" title="event log example" src="http://i1.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2010/02/event-log-examoke.png?resize=483%2C526" alt="" data-recalc-dims="1" /></a>
</p>

**Why Do It?**

I was disappointed with the built in trace listener provided with Enterprise Library.  It would write a begin and end message to the trace utility, and output a rather verbose message containing only a few relevant lines.  Even though it was easy to use in a using() statement, it was rather difficult to view related trace statements for a single high-level call.  I could have explored using the Tracer class more effectively, but isn&#8217;t always easier to write your own?

**How it Works**

Usually you&#8217;ll create an instance of the PerformanceTracer in a using statement, just like the Tracer class.

<pre class="brush: csharp; title: ; notranslate" title="">using(new PerformanceTracer("How long does this function take?") {
     //Your Code Here
}
</pre>

This will write the current message as the Title of the LogEntry and write the elapsed time between instantiation and disposal (thanks to the IDisposable pattern).  But the real benefit comes when you use the instantiated class for tracing within the using block:

<pre class="brush: csharp; title: ; notranslate" title="">using(var tracer = new PerformanceTracer("Some long process") {
    //Step One
    tracer.WriteElapsedTime("Step One Complete");
    //Step Two
    tracer.WriteElapsedTime("Step Two Complete");
}
</pre>

This will write both the elapsed time and the difference between statement calls, and will allow you to easily gain insight into long running steps.  There&#8217;s also another method _WriteInfo_, which allows you to write important information without clogging the performance messages.  This is important for information such as the current user, or information about the request:

<pre class="brush: csharp; title: ; notranslate" title="">tracer.WriteInfo("CurrentUser", Identity.User);
tracer.WriteInfo("QueryString", Request.QueryString);
</pre>

More often than not you probably differ high level function calls (those that orchestrate complex logic) to sub routines.  In Domain Driven Design, your Domain objects will probably interact with support services or other entities.  You may need to gain insight into these routines, but you don&#8217;t want to create separate tracers- that will create multiple EventLog entries and won&#8217;t give you a clear picture of the entire process.  In a production environment, where there could be hundreds of concurrent requests, aggregating those calls in a meaningful way is a nightmare.  That&#8217;s the main issue I had with the default Tracer class.  The solution for the PerformanceTracer is simple-  In your dependent classes you create a property of type IPerformanceTracer.  There are two extension methods, _LogPerformanceTrace_ and _LogInfoTrace_ you can use dependent classes.  These simply do a null object check before writing the trace so you don&#8217;t get any nasty null reference errors- helpful if you need to add logging and don&#8217;t want to update dependencies in your unit tests.  Notice how in the example above there was one call which took the majority of the time? I should probably get more insight into that function to see what&#8217;s going on.  Here&#8217;s an example of a property/extension method combination:

<pre class="brush: csharp; title: ; notranslate" title="">//Create a property to store the tracer
public IPerformanceTracer Tracer { get; set; }
public void MySubRoutineCalledFromAnotherFunction(string someParam)
{
    //Do Work
    Tracer.LogPerformanceTracer("Subroutine Step One");
    //Do More Work
    Tracer.LogPerformanceTrace("Subroutine Step Two");
}
</pre>

It doesn&#8217;t matter if the Tracer property is set, because the extension method handles null objects correctly. You could also put the tracer entity in an IoC framework and pull it out that way.  I prefer this approach when dealing with web apps.  You can create the tracer in a BeginRequest HttpModule, plug it into a container, and pull it out in the EndRequest method and dispose of it.  Using the tracer with a Per WebRequest Lifetime Container, supported with most frameworks, is even better.  This way it&#8217;s available everywhere without have to wire up new properties, and you can scatter it around in various places.  Here&#8217;s some code which pulls it out of a ServiceLocator ActionFilterAttribute in a ASP.NET MVC App:

<pre class="brush: csharp; title: ; notranslate" title="">public class PerformanceCounterAttribute : ActionFilterAttribute

{

IPerformanceTracer tracer;

public override void OnActionExecuting(ActionExecutingContext filterContext)

{

     tracer = ServiceLocator.Current.GetInstance&lt;IPerformanceTracer&gt;();

     tracer.LogPerformanceTrace("Action Executing");

}

public override void OnResultExecuted(ResultExecutedContext filterContext)

{

     tracer.LogPerformanceTrace("Result Finished");

}

public override void OnActionExecuted(ActionExecutedContext filterContext)

{

     tracer.LogPerformanceTrace("Action Finished");

}

}

</pre>

The final aspect of this setup is formatting the output.  EnterpriseLibrary provides a ton of information about the the context which logged the entry- not all of it may be applicable to you.  I use a customer formatter to only write the Title, Message and Category of the trace, like so:

<pre class="brush: xml; title: ; notranslate" title="">&lt;formatters&gt;
 &lt;add template="Title:{title}&#xA;Message: {message}&#xA;Category: {category}" type="Microsoft.Practices.EnterpriseLibrary.Logging.Formatters.TextFormatter, Microsoft.Practices.EnterpriseLibrary.Logging, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" name="Text Formatter"/&gt;
 &lt;/formatters&gt;
</pre>

This makes entries so much more readable. By default, the EventLog takes care of important information like the Computer name and Timestamp. So I can keep the message info very clean.

﻿One thing I&#8217;m specifically not trying to do is get an average of the same function call over a period of time.  That insight is also very important in understanding bottlenecks.  But what I&#8217;m trying to do is get a broad sense of the flow of a large process and see which chunks can be improved.  With the ability to aggregate EventLogs in a single place, I can easily sift through and see how my app is behaving in production.  By linking that with relevant information like the current user, url, and query string, I can find specific scenarios which are problematic that may not have gotten caught in lower environments.

A copy of the entire class is below, and available from GitHub.  It doesn&#8217;t make sense to roll out an entire project for it- just roll it somewhere in your app and tweak as necessary.  Then enjoy finding all the places you could optimize!

 [1]: http://gist.github.com/302225
