---
title: Organizing Javascript for Event Pooling with jQuery
author: Michael
type: post
date: 2009-12-21
url: /2009/12/organizing-javascript-for-event-pooling-with-jquery/
dsq_thread_id:
  - 527224558
categories:
  - Programming
tags:
  - designpatterns
  - jquery
---
It turns out my most popular article of the past year was [Event Pooling with jQuery&#8217;s Bind and Trigger][1].&nbsp; I wanted to write a follow up article taking this approach one step further by discussing how to logically organize the relationship between binders and triggers on a javascript heavy UI.&nbsp; It&#8217;s important to properly design the code structure of your javascript to create a flexible and maintainable system.&nbsp; This is essential for any software application. &nbsp;For javascript development, you don&#8217;t want to end up with odd dependencies hindering changes or randomly bubbled events causing bugs.

### Event Pooling: A Quick Review

What is event pooling and why is it important?&nbsp; Quite simply it&#8217;s a way to manage dependencies.&nbsp; You create a loosely coupled system between the thing which triggers an action to happen and the thing which responds to the action, called the binder.&nbsp; jQuery has some cool bind and trigger functionality which allows you to create custom events for event pooling- and you can use this to easily wire up multiple functions to write complex javascript with ease.&nbsp; I encourage you to check out the original how-to article [Event Pooling with jQuery&#8217;s Bind and Trigger][2] to learn more. &nbsp;It&#8217;s a very powerful technique.

Now, as we all know, with great power comes great responsibility.&nbsp; If you structure the relationship between binders and triggers incorrectly you&#8217;ll end up with a mess on your hands, a spider web of logic which is almost impossible to unwind.&nbsp; Here are some tips to better structure your binders and triggers for a logical and maintainable application

### Presenting the Problem

[<img class="alignleft size-medium wp-image-292" title="Stupid Form" src="http://i1.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2009/12/Stupid-Form1-300x179.png?resize=300%2C179" alt="Stupid Form" data-recalc-dims="1" />][3]

Let&#8217;s take a look at an example app. &nbsp;Here&#8217;s a simple form where a user can draft a report, save it for later, or send it immediately. There are three options to enter data in the name/email/department field.&nbsp; The user can enter it manually, choose from the autosuggest area at left, or simply select one of the favorite links.&nbsp; The autosuggest/favorite link would pre-populate the textboxes used in the form. &nbsp;If all the data is there, the send button is enabled. &nbsp;If not, it&#8217;s disabled.

Let&#8217;s discuss how we&#8217;d deal with enabling and disabling the send button. &nbsp;There are many ways to approach this. &nbsp;A traditional way is to wire up some standard event to our textboxes:

<pre class="brush: jscript; title: ; notranslate" title="">$('#name').keyup(function(){
	//Logic (hopefully structured in a reusable way across all controls).
});
//Do this for all other input controls.
</pre>

This solution is perfectly valid and works, but it has some shortcomings. &nbsp;First, you need to wire up all three controls and route them to the same function. &nbsp;Next, even though the keyup will handle user input, the textboxes could change in other ways- either from the favorite link or the auto suggest box. &nbsp;We&#8217;ll need to call our validation function in numerous places. &nbsp;There may also be other logic we want to incorporate within the text change event unrelated to enabling of the send button- email validation, for instance, which is only applicable to the email textbox, or a certain length requirement on the name textbox. &nbsp;What happens if there&#8217;s a new requirement where the subject/body must be filled to enable the send button? Where does all this functionality fall into our larger requirements?

You can imagine the complexity of writing the javascript required for this UI. &nbsp;We need a slew of functions to deal with validation logic: Both for enabling the send button and other input controls. &nbsp;We need to wire up all our onkeyup and on click events across the auto suggest field, textboxes, and favorite links. &nbsp;Function calls are everywhere. &nbsp;It will take all your code complete skills to manage those dependencies and keep this code lean.

This is where event pooling comes into play: instead of direct dependencies between these controls and logic, you bubble events to a middle man and allow interested parties to respond accordingly. &nbsp;Instead of all the controls telling the send button to enable/disable itself, the send button &#8220;listens&#8221; for changes it cares about- when the value of the textbox changes- and responds accordingly. &nbsp;The responsibility is reversed- the function which needs to be called can choose when it&#8217;s called, rather than waiting for something to call it.

### Dependencies Between Binders and Triggers

With event pooling there are two things you need to be conscience of:&nbsp; the events themselves and the data passed between the binder and trigger.&nbsp; These items form the two dependencies between binders and triggers.&nbsp; The event serves as a link between the binder and trigger and the data is the information which is passed between the two.&nbsp; It&#8217;s essential to properly structure your events and choose the right data transfer option to avoid pitfalls over the life of the application.&nbsp; We&#8217;ll deal with each aspect individually. &nbsp;This post will be about the structure of events, and I&#8217;ll write various ways to pass data between parties in another post.

### Structuring Custom Events

<span style="font-weight: normal;">Most people are familiar with the standard javascript events: click, blur, enter, exit, etc.&nbsp; These are what you usually wire up to functions when you want to do something.&nbsp; However, they only go so far- you need custom events when you have a lot of stuff happening. &nbsp;Why use them?&nbsp; Quite simply, it&#8217;s more logical for your application control flow. &nbsp;For our send button functionality, we want our validation function to act when something happens. &nbsp;This &#8220;something&#8221; will be a custom event we create. &nbsp;We have two options for naming our event: we can name the event after what is intended, like &#8220;ENABLE_DISABLE_SEND_BUTTON&#8221;, or name the event after what has happened, like &#8220;NAME_CHANGED&#8221; or &#8220;FAVORITE_LINK_SELECTED&#8221;. &nbsp;The former option requires multiple events to be fired when something happens. &nbsp;The autosuggest box, for instance, would require the ENABLE_DISABLE_SEND_BUTTON trigger, a SELECTED_CONTACT event for setting the textboxes, and whatever else happens after a contact is selected. &nbsp;The latter option requires just one trigger to be fired, but the binder must subscribe to multiple events. &nbsp;The choice can also be presented like this: do we want each element to fire a trigger for each action which should occur, or have a function to listen for multiple triggered events?</span>

<span style="font-weight: normal;"><a href="http://i0.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2009/12/Event_Pooling-2-e1261347350823.png"><img class="aligncenter size-full wp-image-300" title="Event_Pooling-2" src="http://i0.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2009/12/Event_Pooling-2-e1261347350823.png?resize=600%2C317" alt="" data-recalc-dims="1" /></a><br /> </span>

Naming events may seem like an arbitrary and thoughtless act, but it&#8217;s essential to have a naming strategy with event pooling. &nbsp;Just like wiring up functions directly can be unwieldy, so can a slew of events wired up every which way.

**For event pooling, it&#8217;s more important to name events after what has happened, rather than what is intended.** So the correct approach is to go with things like &#8220;NAME\_CHANGED&#8221;, &#8220;CONTACT\_SELECTED&#8221;, or &#8220;EMAIL\_CHANGED&#8221;. &nbsp;The rationale has to do with dependencies themselves: &nbsp;we don&#8217;t want functions called from random parts of the system via specific events: &nbsp;this is no better than calling functions directly from disparate parts of the system. &nbsp;And with specific action related events like &#8220;ENABLE\_SEND&#8221; you just know someone is going to take a shortcut and wire some random binder to a totally unrelated trigger, because that trigger is fired from the control they want to monitor. &nbsp;Binders- and the functions they are wired to- should proactively &#8220;listen&#8221; for things which it&#8217;s interested in. &nbsp;This allows you to easily know why a function is being called. &nbsp;If you need to change why or how something is handled, you go to the recipient, not the caller. &nbsp;The caller, after all, could be anything, and more importantly the caller could be triggering multiple things.

<pre class="brush: jscript; title: ; notranslate" title="">//Funnel all possibilities to single custom trigger
$('#name').bind('keyup enter', function() {
 $(document).trigger('NAME_CHANGED');
});

//Funnel all possibilites to single custom trigger
$('#email').bind('keyup enter', function() {
$(document).trigger('EMAIL_CHANGED');
});

//I know why this is being called because I've subscribed
//to the events I'm interested in.
$(document).bind('NAME_CHANGED EMAIL_CHANGED', function()
{
 //Handle validation, etc.
});
</pre>

The cool thing about this approach is there may be something else which will cause an email or name changed event to happen: specifically, when someone has selected a contact from the autosuggest list or a favorite link is selected. &nbsp;You can fire the name_changed event from multiple places and not worry about wiring up any new triggers. &nbsp;The code would look something like this:

<pre class="brush: jscript; title: ; notranslate" title="">//Fired from elsewhere
$(document).bind('FAVORITE_LINK_SELECTED', function()
{
  //Handle selection
  //Set Name, Email, etc.
  //Fire event:
  $(document).trigger('EMAIL_CHANGED');
  $(document).trigger('NAME_CHANGED');
});
</pre>

How nice: &nbsp;another part of the system changes the name indirectly, and we don&#8217;t have to worry about hooking anything up because the binder is already subscribed to the NAME_CHANGED event.

Note the name doesn&#8217;t necessarily correspond to a specific element: It&#8217;s not a textbox\_name\_changed event, nor are any specific html id&#8217;s involved except for wiring up a trigger from a standard keyup event. &nbsp;This is an important difference: we could rename the id&#8217;s, or switch to some other input control, and not have to rewire everything. &nbsp;We don&#8217;t care about the textbox nor that the textbox&#8217;s value has changed. &nbsp;We care that the textbox respresents the name entered or the email address- and we want to know when the name or email has changed. &nbsp;The favorite link clicked is a good example of this nuanced difference. &nbsp;Take a look at the following html:

<pre class="brush: xml; title: ; notranslate" title="">&lt;a href='#' id='fav1' class='favorite'&gt;My Favorite 1&lt;/a&gt;
&lt;a href='#' id='fav2' class='favorite'&gt;My Favorite 2&lt;/a&gt;
</pre>

Imagine we&#8217;ve wired the click event to fire a FAVORITE\_LINK\_SELECTED trigger. &nbsp;With this trigger, we don&#8217;t care that the link with id fav1 or fav2 has been clicked, nor the a.favorite selector has been clicked. &nbsp;We don&#8217;t care about ids or css classes. &nbsp;We care a FAVORITE\_LINK\_SELECTED event has happened because that&#8217;s what the id fav1 and favorite class represents- the favorite link- and we want to know when that has been selected. &nbsp;We can rename the id, change the class, or even change the entire element. &nbsp;As long as FAVORITE\_LINK\_SELECTED is fired we&#8217;re good to go. &nbsp;Our custom FAVORITE\_LINK\_SELECTED trigger is the abstraction which creates the loosely coupled system.

### Conclusion

One thing I haven&#8217;t discuss is how data is passed from trigger to binder. &nbsp;It&#8217;s the other important dependency between triggers and binders which we&#8217;ll discuss in another post. &nbsp;The important thing to take away is why you&#8217;d want to use custom named events to create a loosely coupled system. &nbsp;For very straightforward pages it&#8217;s probably not worth the overhead- the abstraction isn&#8217;t required. &nbsp;However, in a complex form where there are lots of validation dependencies, or many routes to the same function, or when multiple events can trigger an update- you want to use event pooling. &nbsp;More importantly, you want to think about your strategy when it comes to naming events, because it&#8217;s the description linking the caller and method together. &nbsp;You do not want to code yourself into a corner!

 [1]: http://www.michaelhamrah.com/blog/index.php/2008/12/event-pooling-with-jquery-using-bind-and-trigger-managing-complex-javascript/
 [2]: ../index.php/2008/12/event-pooling-with-jquery-using-bind-and-trigger-managing-complex-javascript/
 [3]: http://i2.wp.com/www.michaelhamrah.com/blog/wp-content/uploads/2009/12/Stupid-Form1.png
