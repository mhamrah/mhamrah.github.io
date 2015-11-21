---
title: 'Event Pooling with jQuery Using Bind and Trigger: Managing Complex Javascript'
author: Michael
type: post
date: 2008-12-05
url: /2008/12/event-pooling-with-jquery-using-bind-and-trigger-managing-complex-javascript/
dsq_thread_id:
  - 527224432
categories:
  - Programming
tags:
  - designpatterns
  - jquery
---
**Managing Complexity in the UI**

As everyone knows, the more dependencies you have in a system, the harder maintaining that system is.  Javascript is no exception- and orchestrating actions across complex user interfaces can be a nightmare if not done properly.

Luckily, there&#8217;s a great pattern for orchestrating complex interaction in a disconnected way. No, it&#8217;s not the Observer pattern.  It&#8217;s a take on the Observer pattern called Event Pooling which is a piece of cake with jQuery&#8217;s [bind][1] and [trigger][2] functions.  <a title="jQuery bind and trigger example" href="http://www.michaelhamrah.com/blog/wp-content/uploads/jqueryEventPool/index.html" target="_blank">For the get to the code folks, here&#8217;s an example of using jQuery&#8217;s bind and trigger for event pooling</a>.

**Problems with the Observer Pattern**

The observer pattern is great for some things, but it still requires a dependency between the observer and the subject.  The publish/subscribe scenario creates a direct relationship between two objects- and makes orchestrating a lot of events difficult when you have to manage so many direct references.<figure id="attachment_56" style="width: 300px;" class="wp-caption aligncenter">

[<img class="size-medium wp-image-56" title="observer pattern" src="http://i0.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2008/12/observer-300x91.png?resize=300%2C91" alt="observer pattern" data-recalc-dims="1" />][3]<figcaption class="wp-caption-text">observer pattern</figcaption></figure> 

**Event Pooling**

Event pooling is simply a variation on the Observer pattern, where a &#8220;middle man&#8221; is used to orchestrate the publish/subscribe system.  First, an observer will register with the event pool by saying &#8220;I need to call this function when this event is fired&#8221;.  Next, the subject will tell the event pool &#8220;I&#8217;m firing this event&#8221;.  Finally, the event pool will call the function the observer registered on behalf of the subject.  The observer and subject only need to know about the event pool, not each other.<figure id="attachment_57" style="width: 300px;" class="wp-caption aligncenter">

[<img class="size-medium wp-image-57" title="event-pool" src="http://i2.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2008/12/event-pool-300x147.png?resize=300%2C147" alt="Event Pool" data-recalc-dims="1" />][4]<figcaption class="wp-caption-text">Event Pool</figcaption></figure> 

This provides some cool functionality- especially because if you need to reference the subject (called the publisher), you can get a direct reference via the event pool.  This is similar to the object sender parameter in .NET&#8217;s event system.

**Show me Code!**

We are going to do two things: First, update a span tag to show an address when a user enters a name, city, or state.  Second, show some complex behavior by daisy changing events and binding a single function to multiple triggers. <a title="jQuery bind and trigger example" href="http://www.michaelhamrah.com/blog/wp-content/uploads/jqueryEventPool/index.html" target="_blank">Check out the example.</a>

We have an UpdateOutput() function that updates the span:

<pre class="syntax js">function UpdateOutput() {
    var name = $('#txtName').val();
    var address = $('#txtAddress').val();
    var city = $('#txtCity').val();

    $('#output').html(name + ' ' + address + ' ' + city);
}
</pre>

This is wired up via the bind function:

<pre class="syntax js">$(document).bind('NAME_CHANGE ADDRESS_CHANGE CITY_CHANGE', function() {
    UpdateOutput();
});
</pre>

Notice how we&#8217;re wiring up multiple event names with a single function which calls UpdateOutput.  This allows us to encapsulate common functionality in a single function which could be called from various sources.

We&#8217;re also binding and firing to $(document).  Document provides a global bucket all functionality has access to- a static class that can be referenced from anywhere, making pooling easy.

We can also wire up another function with the NAME_CHANGE event without effecting any other logic:

<pre class="syntax js">$(document).bind('NAME_CHANGE', function(e) {
    UpdateName();
    UpdateOtherText(e);
});
</pre>

Here, NAME_CHANGE will also trigger UpdateName and UpdateOther.

Firing an event is done with the trigger function:

<pre class="syntax js">$('#txtAddress').keyup(function() {
    $(document).trigger('ADDRESS_CHANGE');
});
$('#txtCity').keyup(function() {
    $(document).trigger('CITY_CHANGE');
});
</pre>

It can also be called directly from html:

<pre class="syntax html escaped">Name: &lt;input type="text" id="txtName" onkeyup="javascript:$(document).trigger('NAME_CHANGE');" /&gt;
</pre>

**Advanced Functionality**

There are two more things you can do with bind and trigger: Inspect who fired the trigger (this can be important if you&#8217;re wiring up a single function from multiple subjects) and pass data to the event.

In our next example, our function definition takes in two parameters: e, which is the event&#8217;s sender, and data, a generic data bucket:

<pre class="syntax js">$(document).bind('FIRST_UPDATED', function(e, data) {
    UpdateOtherText(e, data);
});
</pre>

The first parameter will always be the event&#8217;s sender.  You can wire up as many other parameters as you want.  You pass data as a comma delimited list between brackets [parm1,parm2] when calling trigger:

<pre class="syntax js">$(document).trigger('FIRST_UPDATED', [data]);
</pre>

and from html:

<pre class="syntax js escaped">&lt;input type="text" id="txtData" onkeyup="javascript:$(document).trigger('DATA_CHANGED', [$(this).val()]);" /&gt;
</pre>

The sender object has a couple of properties, but the most important is &#8216;type&#8217;, which let&#8217;s you know what triggered the event:

<pre class="syntax js">function UpdateOtherText(e, text) {
var text;
if (e.type == 'FIRST_UPDATED')
 text = 'from first: ' + text;
else
 text = e.type;
$('#eventFired').html(text);
}
</pre>

Check out my follow up article [Organizing Events for Event Pooling.][5]

[][5]
  
[<img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2008%2f12%2fevent-pooling-with-jquery-using-bind-and-trigger-managing-complex-javascript%2f&bgcolor=0000FF" border="0" alt="kick it on DotNetKicks.com" />][6]

 [1]: http://docs.jquery.com/Events/bind "jQuery Bind"
 [2]: http://docs.jquery.com/Events/trigger "jQuery Trigger"
 [3]: http://i2.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2008/12/observer.png
 [4]: http://i2.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2008/12/event-pool.png
 [5]: http://www.michaelhamrah.com/blog/index.php/2009/12/organizing-javascript-for-event-pooling-with-jquery/
 [6]: http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2008%2f12%2fevent-pooling-with-jquery-using-bind-and-trigger-managing-complex-javascript%2f
