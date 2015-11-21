---
title: Saving dynamically added list items using jQuery and ASP.NET MVC
author: Michael
type: post
date: 2008-12-15
url: /2008/12/saving-dynamically-added-list-items-using-jquery-and-aspnet-mvc/
dsq_thread_id:
  - 527224431
categories:
  - Programming
tags:
  - aspnetmvc
  - jquery
---
There are going to be times when you want to allow a user to enter multiple copies of a single form on a web page.  This frequently happens when adding items to a list- like products in a shopping cart or tasks in a task list.  You want the user to add as many &#8220;items&#8221; as they want to the list, then save the entire list at once.

Dynamically adding elements to a page is easy with jQuery, but parsing out list items on the server can be difficult- especially when you don&#8217;t know how many items are on the page!   Things get even trickier when the number of input controls for each item increases- you have to keep all these input controls in sync so each item gets saved correctly.  Luckily, ASP.NET MVC has <a href="http://haacked.com/archive/2008/10/23/model-binding-to-a-list.aspx" target="_blank">a built in feature to pull out a list of complex types posted to a page</a> and automatically put them in a model.  We&#8217;re going to combine this feature with jQuery to dynamically add form elements on a page which end up in a list of complex types that can be accessed in a controller action.

In our example,  a user will be creating a shopping list and will  have the ability to add items to the shopping list dynamically.  When the user hits &#8220;Save&#8221; the entire list with all items will be posted to the server and serialized into a ShoppingList model.

We&#8217;re going to use:

<pre class="syntax html"><ul>
  <li>
    jQuery to dynamically drive the client side and dynamically add list items
  </li>
  	
  
  <li>
    ASP.NET MVC as our web infrastructure
  </li>
  	
  
  <li>
    The DefaultModelBinder to build a list of complex types which can be passed to an action method.
  </li>
  
</ul>
</pre>

Our model is simple- we have a ShoppingList with properties of Name and ShoppingItems:

<pre class="syntax c#">public class ShoppingList
{
public string ListName { get; set; }
public IList&lt;ShoppingItem> Items { get; set; }
}

public class ShoppingItem
{
public string Name { get; set; }
public int Quantity { get; set; }
}

</pre>

**Posting Multiple Items At Once**

The key to this feature is posting multiple related elements which end up in a list.  In order for the DefaultModelBinder to build a list, we have to post elements in a certain way.  Essentially, each set of related input elements are grouped together using a uniquer key.  The key can be any string- it doesn&#8217;t have to be an integer index.

Keys are specified with a hidden input element.  In our sample, we want our list to end up in the myList.Items property.  So for every object in the Items list, we need a hidden input field with a name of &#8220;myList.Items.Index&#8221;.  The key we specify for the element will be the key we use in the name attribute the input element.  If our key is &#8220;foo&#8221;, and our property is &#8220;Quantity&#8221;, we&#8217;ll have:

<pre class="syntax html"><input type="hidden" name="myList.Items.Index" value="foo" />

<input type="hidden" name="myList.Items[foo].Quantity" value="5" />

[/pre]

Even though we couldn't do that in c#, we can do it in our markup.  Think of keys for a hashtable- the key can be anything you want.  The hashtable is a list of the complex type your building, so typing hashtable["myKey'] returns the complex type, and you have access to all properties of that type.

I've created a unit test which shows off this magic for multiple items:



<pre class="syntax c#">

shoppingController.Request.Form.Add("myList.ListName", "My Shopping List");
shoppingController.Request.Form.Add("myList.Items.Index", "1");
shoppingController.Request.Form.Add("myList.Items[1].Name", "Chocolate");
shoppingController.Request.Form.Add("myList.Items[1].Quantity", "5");
shoppingController.Request.Form.Add("myList.Items.Index", "Alpha");
shoppingController.Request.Form.Add("myList.Items[Alpha].Name", "Graham Crackers");
shoppingController.Request.Form.Add("myList.Items[Alpha].Quantity", "10");

</pre>


<p>
  Notice how we're passing multiple "myList.Items.Index" values.  Don't worry- they won't overwrite eachother.  Servers turn multiple name values into a comma delimited list (you'll end up with: "1,Alpha" has your values for "myList.Items.Index".  All the DefaultModelBinder is doing is parsing the list of  keys for whatever array you want, then matching those values to the form package. My naming convention of myList.Items is simply allowing the myList parameter in the controller action to be populated correctly by the DefaultModelBinder.
</p>


<p>
  To reiterate, the indexer I use for each array object is simply grouping related html elements together- Graham Crackers has a quantity of 10 because I'm using the same indexer for each name: Items[Alpha].Name and Items[Alpha].Quantity, just like a hashtable.
</p>


<p>
  The rest of the unit test shows how the DefaultModelBinder will build the ShoppingList Item's property using these form values:
</p>


<pre class="syntax c#">

var defaultBinding = ModelBinders.GetBinder(typeof(ShoppingList));
var bindingContext = new ModelBindingContext(shoppingController.ControllerContext,
shoppingController.ValueProvider,
typeof(ShoppingList),
"myList", null, shoppingController.ModelState, null);
var binderResult = defaultBinding.BindModel(bindingContext);

Assert.IsNotNull(binderResult);
Assert.IsNotNull(binderResult.Value);
Assert.IsInstanceOfType(binderResult.Value, typeof(ShoppingList));

var myList = binderResult.Value as ShoppingList;

Assert.IsTrue(myList.ListName == "My Shopping List");
Assert.IsTrue(myList.Items.Count > 0);
Assert.IsTrue(myList.Items[0].Name == "Chocolate");
Assert.IsTrue(myList.Items[0].Quantity == 5);

Assert.IsTrue(myList.Items[1].Name == "Graham Crackers");
Assert.IsTrue(myList.Items[1].Quantity == 10);

</pre>


<p>
  <strong>The View</strong>
</p>


<p>
  I want a button to "add another item" to the shopping list.  If the user clicks this button, two new textboxes should appear: one for name, and another for quantity.  These need to have the same indexer.  When the page first loads, there should already be an item to enter.
</p>


<p>
  I have a couple of choices to build this logic.  I could specify one set of input elements in the view, and use jQuery to add individual elements to the DOM when the user hits the button.  I don't like this idea because it means I have two places to build the list items: one in the view, the other in jQuery.  When you build the same code in multiple places, you're going to get discrepancies, which lead to bugs-  I guarantee it. I could use jQuery to add the default form when the page loads, but I don't like this- html is easy and simply, it allows me to see what I'm doing.  It's much easier to write html than write javascript to build html.
</p>


<p>
  Instead, I'm going for a partial view approach using actions.  By putting the injected section of markup into its own UserControl with a controller action I can embed the markup in the parent view and use ajax to get the rendered html snippet to the client via a url.  I can also choose how to generate the indexer in the action method's body: I'm going to use Guids.  With Guids, I won't have to track any other indexers- I'm guaranteed a unique value I can use for all the related elements in the partial view.  Here's the markup:
</p>


<p>
  The parent view, which the user sees on load:
</p>


<pre class="syntax html">



</pre>


<p>
  The partial view for each item:
</p>


<pre class="syntax html">



<div class="itemContent">
  &lt;input type="hidden" name="&lt;%= ViewData["Prefix"] + ".Index" %>" value="&lt;%=ViewData["GUID"] %>" />
  <label>Item: </label>&lt;input type="text" name="&lt;%= ViewData["Prefix"]  + "[" + ViewData["GUID"] + "].Name" %>" />
  <br />
  <label>Quantity: </label>&lt;input type="text" name="&lt;%= ViewData["Prefix"] + "[" + ViewData["GUID"] + "].Quantity" %>" />
  <br />
  
</div>

</pre>


<p>
  I chose to make the "prefix" I need a variable so I can potentially use this same view in other forms if needed.  I may want to put this form somewhere else, where the server argument isn't myList, but something else- I could forgo a parameter of ShoppingList and want to post a list of ShoppingItems.  This is useful when adding some sort of "update" feature in another section of page- say, when I already have a ShoppingList and I'm updating with a new list of items.
</p>


<p>
  On the client side I simply wire up my button to request the html snippet from the server and inject that snippet with jQuery:
</p>


<pre class="syntax js">

$(document).ready(function() {
$("#btnAddAnother").click(function() {
$.ajax(
{
type: "GET",
url: "/Shopping/ShoppingItemFormContent/myList.Items",
success: function(result) {
var toInject = $(result);
$("#itemContainer").append(toInject);
}
});

})

});

</pre>


<p>
  There's something I don't like about this: I need to call the server every time I need a view.  This isn't that snappy, and could create a lot of chatter with the server.  There's a way around this for brevity I'm only going to explain: render the indexer as a specific value which can be parsed out and replaced with something else later.  My original goal is only wanting one place to specify markup: I do not want to have to duplicate code across a project.  But that shouldn't mean I need to call the server every time I need an html snippet.  I could make the snippet regular html which can be cached, then use a string or regular expression replace function to replace the hard coded indexer with something unique.
</p>


<p>
  <a href="http://michaelhamrah.com/blog/wp-content/uploads/postingalist.zip">You can download the sample project here.</a><br />
  <a href="http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2008%2f12%2fsaving-dynamically-added-list-items-using-jquery-and-aspnet-mvc%2f"><img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2008%2f12%2fsaving-dynamically-added-list-items-using-jquery-and-aspnet-mvc%2f&#038;bgcolor=0000CC" border="0" alt="kick it on DotNetKicks.com" /></a>
</p>
