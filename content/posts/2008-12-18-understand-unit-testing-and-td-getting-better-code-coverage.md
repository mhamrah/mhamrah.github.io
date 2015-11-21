---
title: 'Understand Unit Testing and TDD: Getting Better Code Coverage'
author: Michael
type: post
date: 2008-12-18
url: /2008/12/understand-unit-testing-and-td-getting-better-code-coverage/
dsq_thread_id:
  - 527224436
categories:
  - Programming
tags:
  - TDD
  - UnitTesting
---
One of the biggest challenges in unleashing the power of unit testing is getting good code coverage.  Most of the time, especially when teams are just starting out with Test Driven Development, unit testing usually gets in the way.  A lot of people (I succumb to this syndrome sometimes) add tests _after_ they develop code.  The audacity! Some people just don&#8217;t unit test at all- especially when struggling with mocking a dependency, like a database or the HttpContext object.  And forget about finding more than one unit test for functions- you already have one for a function which passes- so you&#8217;re done, right?

The secret in unit testing is understanding what you need to test.  You are not just testing something works- you&#8217;re testing specific functionality behaves _as expected_.  Once you understand this difference, not only will TDD be easier to achieve, but developing large projects, ensuring business rules, enforcing API guidelines, and concurrent development on the same feature will be easier and faster for you and your team.

This post will help you think about testing: what you need to do to add unit tests to your existing project or add to what you already have.  We&#8217;re not doing test first- hopefully, this will be a precursor to TDD in getting you thinking about what to get out of your unit tests so you can get into the TDD groove.

**Expectations And Reactions
  
** 

Each class- and each function- has a specific purpose.  Systems are built on the interaction of these classes and functions.  Classes call functions.  Functions call other functions.  Data is processed.  Objects are returned.  Exceptions are thrown.  All of this interaction is like a clock- the pendulum swings, the gears turn, the hands move.  When everything does what it should, you have a healthy, usable, working system.  When something doesn&#8217;t, chaos ensues and thing don&#8217;t work- the hands of the clock don&#8217;t move.

This boils down to a simple concept of expectations and reactions.  Class A expects Class B to do something when Class A calls a function on Class B.  Class A is _dependent_ on Class B&#8217;s behavior.  What you&#8217;re looking to do is create a suite of tests to ensure Class B is actually doing the behavior Class A expects- from valid return values to handling exceptions- so the hands on the clock always turn.

**The Code**

[Download the sample code here.][1]

I&#8217;ve created some simple code which represents an ordering system we&#8217;ll say is part of typical three tier MVC application, similar to <a href="http://blog.wekeroad.com/mvc-storefront/" target="_blank">Rob Conery&#8217;s MVC Storefront</a>, I&#8217;ve backed everything with interfaces.  To keep things simple, I&#8217;ve done this as a single class library- you can imagine all the other mumbo jumbo thrown in there.

We&#8217;re simply going to be shipping an order- which involves getting an order from our data tier and then using a shipping service to ship the order.  For this series, I&#8217;m only going to focus on a single method in our business tier: the OrderShipmentManager.ShipOrder() function.  This will help us look at one of the two main components for a successful test suite: knowing what to tests and setting up mock objects.  In a real world application, you&#8217;ll want to apply the expectation/reaction approach to each layer.

Currently, I&#8217;m only going to focus on what tests we need to have.  I&#8217;ll dig into how to set up mocking in another post.  Let&#8217;s start by looking at how our ShipOrder is being called in the controller:

IOrderShipmentManager opm = new OrderShipmentManager(new DataTier.OrderStorage(), new DataTier.ShipmentService());
  
Shipment shipment = null;

try
  
{
  
shipment = opm.ShipOrder(orderId);
  
}
  
catch (OrderShipmentException ope)
  
{
  
System.Diagnostics.Debug.Write(&#8220;Whoops: {0}&#8221;, ope.Message);
  
}

View(shipment);

Most controllers should look like this: very light, with the bulk of your application in your business logic layer.  This is a fairly simple call: in my controller, I take in an orderId I want to ship, and pass it to ShipOrder(). The controller expects two things to happen: a shipment to be returned, which it can pass to the view; or an exception to be thrown.

So let&#8217;s make sure we have two unit tests to cover these two expectations (Note: I always prefix unit tests with CLASS\_FUNCTION.  This makes viewing test results easier. I didn&#8217;t add that here because of redundancy- but the test project will have each test prefixed with OrderShipmentManager\_ShipOrder):

[TestMethod()]
  
public void Returns\_Shipment\_Object()
  
{
  
}
  
[TestMethod()]
  
public void Throws\_OrderShipmentException\_When_What()
  
{

}

So I have my two unit tests- but the problem is, when should ShipOrder return a shipment? What should it actually do? And when does ShipOrder throw an exception?  What the hell do I write in these tests? We could look at the view to figure out what it does with the Shipment object.  We could read the requirements to figure out what ShipOrder should do (or, in an agile world, figure this out from our user story).  But we don&#8217;t have these things.  So let&#8217;s look at the function to see if we can make heads or tells of this:

public Shipment ShipOrder(int orderId)
  
{
  
Order order;
  
Shipment shipment;

if (orderId < 0) throw new OrderShipmentException("OrderId cannot be less than zero"); order = \_orderStorage.GetOrder(orderId); if (order == null) return null; if (order.ShipmentStatus == ShipmentStatus.Shipped) { throw new OrderShipmentException("Can't ship an order that's shipped!"); } shipment = \_shipmentService.CreateShipment(order.Customer.CustomerId); shipment.ShipmentProducts = new List<Product>();
  
order.OrderItems.ForEach(oi => shipment.ShipmentProducts.Add(oi.Product));

_shipmentService.CalculateShipment(shipment);
  
_shipmentService.Ship(shipment);

order.ShipmentStatus = ShipmentStatus.Shipped;
  
order.ShipmentId = shipment.ShipmentId;

_orderStorage.SaveOrder(order);

return shipment;
  
}

Ahh, I get it now! We get the order from our order storage.  If it&#8217;s null, we can&#8217;t ship, so we return null- n0 need for an exception.  If the order&#8217;s been shipped, throw an exception.  Next, create a shipment with our shipment service, calculate the shipment cost, and save the order.  Return the shipment and we&#8217;re done.  Now we can add a suite a tests.

Let&#8217;s focus on exceptions.  Here, we plan on throwing exceptions when either the orderId is bad or the order has already been shipped.  Other code will expect an exception to be thrown under these circumstances.  If we change this logic, things could get screwed up- code can&#8217;t react accordingly, and chaos ensues.  People may have already written code based on these expectations.  So let&#8217;s write some unit tests to ensure we&#8217;re always throwing an exception under these circumstances.

[TestMethod()]
  
public void Throws\_OrderShipmentException\_When\_OrderId\_Is\_Less\_Than_Zero()
  
{
  
}

[TestMethod()]
  
public void Throws\_OrderShipmentException\_When\_Order\_Status\_Is\_Shipped()
  
{
  
}

We&#8217;ve essentially enforced two business rules with these unit tests: when to throw our two exceptions.  (You can argue when or when not to throw an exception.  Personally, I follow the rule: If a function can&#8217;t do what it should, throw an exception).

Coincidentally, we&#8217;re also guaranteeing something else: that our ShipOrder function will always react the same way under certain circumstances.  Here, ShipOrder will always throw an exception with an invalid orderId.  It won&#8217;t return null- and it won&#8217;t continue.  That&#8217;s important for processing logic- we wouldn&#8217;t want to create a shipment when we don&#8217;t have an order.

Next, let&#8217;s go into functional logic.  I want to make sure a couple of things are happening, mainly that the workflow is consistent.  Let&#8217;s add some unit tests around the order object:

[TestMethod()]
  
public void Gets\_Order\_From_Storage()
  
{
  
}
  
[TestMethod()]
  
public void Saves\_Order\_With\_ShipmentId\_And_Status()
  
{
  
}

[TestMethod()]

public void Returns\_Null\_When\_No\_Order()

{
  
}

Next, our shipment object does a couple of more things:

[TestMethod()]
  
public void OrderShipmentManager\_ShipOrder\_Creates\_Shipment\_For_Customer()
  
{
  
}
  
[TestMethod()]
  
public void OrderShipmentManager\_ShipOrder\_Copies\_OrderItems\_To_Shipment()
  
{
  
}
  
[TestMethod()]
  
public void OrderShipmentManager\_ShipOrder\_Calculates_Shipment()
  
{
  
}
  
[TestMethod()]
  
public void OrderShipmentManager\_ShipOrder\_Ships_Product()
  
{
  
}

Finally, we now know what this function should return:

[TestMethod()]
  
public void OrderShipmentManager\_ShipOrder\_Returns\_New\_Shipment\_With\_Order_Items()
  
{
  
}

We have a pretty robust suite of tests around our ship order function.  We&#8217;re not only testing how the controller deals with this function, but the internal workings of the function as well- the expectations other objects have of our ShipOrder method.

**What This Means**

The core principal of testing is to make sure something works as expected.  In principal that&#8217;s totally obvious and makes perfect sense- why the hell am I even writing about it?  In reality, implementing something which meets that definition is extremely vague and easily debatable.  The pitfall most people get into in attempting TDD- or even writing unit tests- is their definition of &#8220;I&#8217;m proving it works as expected&#8221; is wrong.

Evolving systems- especially those built with agile development- change a great deal over time. What we want to do is make sure the behavior of everything we write is always the same as these changes occur.  Ideally, nothing can be taken for granted- return values, exceptions, processing.  All must be tested.  Changes create little ripples in the interaction between all the dependencies in a system. Unchecked, ripples grow bigger and bigger into tsunamis of horrible, horrible bugs.  It&#8217;s important to understand what these interactions are, and make sure new changes don&#8217;t alter current behavior.

**Where TDD Fits In**

TDD simply makes you think about these interactions before you code, so you know what each class and function does for the greater good.  For most people, especially getting started with TDD, that&#8217;s a lot to wrap your head around.  Especially when you&#8217;re the type of coder that likes to code first and have the system morph into what you need it to be.  With TDD, you can easily end up over analyzing the situation, testing too much or too little.  That&#8217;s why TDD is very iterative- figure out what you need, write your single test to enforce it, then write code to pass your test.  But it&#8217;s important to note the level you need from your tests.  If you need a ShipOrder function that returns a shipment, don&#8217;t just check it returns a shipment.  Enforce it&#8217;s doing every step it should- your unit tests are guaranteeing every assumption every other piece of code makes is valid.

**Up Next**

I didn&#8217;t talk about mocking at all.  Mocking is the second most important factor of testing: after knowing what to test!  In another post, I&#8217;ll talk about how to write the unit tests I outlined above.  Unit tests also provide numerous benefits I mentioned earlier in this post: enforcing standards, allowing concurrent development and faster coding (no need for those console apps for class library development!).

[<img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2008%2f12%2funderstand-unit-testing-and-td-getting-better-code-coverage%2f&#038;bgcolor=0000FF" border="0" alt="kick it on DotNetKicks.com" />][2]

 [1]: http://michaelhamrah.com/blog/wp-content/uploads/unittestexample.zip
 [2]: http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2008%2f12%2funderstand-unit-testing-and-td-getting-better-code-coverage%2f
