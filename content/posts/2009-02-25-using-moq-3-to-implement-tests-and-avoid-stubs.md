---
title: Using Moq to Implement Tests (and Avoid Stubs)
author: Michael
type: post
date: 2009-02-25
url: /2009/02/using-moq-3-to-implement-tests-and-avoid-stubs/
dsq_thread_id:
  - 527224621
categories:
  - Programming
tags:
  - moq
  - TDD
  - UnitTesting
---
In [my previous post on understanding TDD][1] I discussed how to analyze existing code for creating unit tests.  This was somewhat of &#8220;reverse TDD&#8221;, the idea being to look for what to needs to be tested- the relationship between what one class expects and what another class actually does.  Unfortunately I stopped short of actually implementing those tests- which is the subject of this post.

As a quick refresher, our demo app has some simple functionality which involves getting an order from a data tier then uses a shipping service to ship the order.  You can [download the sample app here][2].

### Prefer Mocking Over Stubs

Had I written this post about two months ago, I would have favored creating a suite of stub classes to mimic the expected behavior of our interfaces.  The approach would involve creating various stub classes for our dependent interfaces- the IShippingService and IOrderRepository- and plugging those stubs into the various unit tests.  In the simplest sense these would be hard coded classes to do what I wanted- from throwing exceptions to hard coding return values.  Probably even use some fancy random number generation for ids. However, I&#8217;m learning from a current project that the stub approach isn&#8217;t the best route to take.

#### The Problem, Distilled

Stubbing is definitely easy, but code quickly spirals out of control.  With stub classes, especially those backed by interfaces, more classes are created which then need to be managed and supported.  In an evolving software project this management creates added infrastructure which gets in the way when making changes to the original functionality.  An interface change, for example (either adding a new method or refactoring parameters) causes numerous changes to be made in stubbed out classes.  What&#8217;s worse, there can easily be a divergence in the behavior of the stubbed out class with the behavior it&#8217;s supposed to represent.  True, this can happen with mocking, but as we&#8217;ll see mocking is much more granular and can be tailored to specific circumstances with minimal code duplication.

There&#8217;s simply no need to create extensive, or multiple, stub objects with all those great mocking frameworks around.  Most mocking frameworks offer a ton of features creating granular control of expectations, return values, parameter verification, and method invocation checking.  I strongly recommend reading Martin Fowler&#8217;s [Mocks Aren&#8217;t Stubs][3] article for more information on the difference between mocks and stubs.

### Using Moq to Mimic Behavior

We&#8217;ll use [Moq][4] 3 to demo what you can do with mocking.  Let&#8217;s start off with our first unit test, ShipOrder\_Returns\_New\_Shipment\_With\_Order\_Items.  We simply want to call our ShipOrder function and ensure we get a shipment back with shipment items.  To begin with, we&#8217;ll use [Moq][4] to mock our IOrderStorage and IShipmentService so we can construct our OrderShipmentManager.

<pre class="syntax c#">[TestMethod()]
public void OrderShipmentManager_ShipOrder_Returns_New_Shipment_With_Shipment_Items()
{
var mockOrderStorage = new Mock&lt;IOrderStorage>();
var mockShipmentService = new Mock&lt;IShipmentService>();

var osm = new OrderShipmentManager(mockOrderStorage.Object, mockShipmentService.Object);

var shipment = osm.ShipOrder(5);

Assert.IsNotNull(shipment, "Returned Shipment was not null");
Assert.IsInstanceOfType(shipment, typeof(Shipment), "Returned object was not of type shipment.");
Assert.IsNotNull(shipment.ShipmentProducts, "ShipmentProducts were null");
Assert.IsTrue(shipment.ShipmentProducts.Count > 0, "ShipmentProducts were empty");
}
</pre>

This test will fail because we haven&#8217;t specified any functionality for our interfaces!  But, we didn&#8217;t get a failing test with a null reference exception when calling GetOrder()- we got a failing test because our returned shipment object from GetOrder was null.  Stepping through code you&#8217;ll see that Moq returned void when calling the IOrderStorage.GetOrder method.  Moq creates a simple &#8220;dumb&#8221; proxy class for the interface automatically.  You can specify a strict behavior using MockBehavior.Strict in the constructor to throw an exception for anything that isn&#8217;t explicitly set up.  This can be helpful for ensuring control flow.

The solution to pass our test is to explicitly tell Moq what to do when this method is called, like so:

<pre class="syntax c#">mockOrderStorage.Setup(os => os.GetOrder(It.IsAny&lt;int>())).Returns((int id) => {
var order = new Order() { OrderId = id };
return order;
});
</pre>

When working with Moq you may struggle with the extensive use of Lambda expressions the framework requires.  But after a couple of tests you&#8217;ll become a lambda ninja.  Moq relies on a pair of Setup/Return calls to dictate behavior.  Also notice how we didn&#8217;t have to implement every method in our interfaces- we only had to implement the ones we needed to make the test pass.  That saves a lot of code from being written!

The previous expression states &#8220;when you call GetOrder, with any int value, return a new order with the input id&#8221;.  Moq gives a lot of power in dictating the behavior of the return call based on the expected function.  By specifing (int id) in our return Lambda we can get a reference to our input parameter, which helps us construct a valid Order object based on any input. This is helpful when setting up exceptions.  By using a predicate expression with It.Is instead of It.IsAny, we can have our call throw an exception for an invalid input:

<pre class="syntax c#">mockShipmentService.Setup(ss => ss.CreateShipment(It.Is&lt;int>(id => id &lt; 0))).Throws&lt;ArgumentException>();
</pre>

Setting up exceptions using Moq is instrumental in properly dealing with exception handling in your code.  Also, using It.Is to tailor return methods can help out when dealing with validation or verifying state.  It&#8217;s much faster and easier to use Moq to create behavior than hard coding stub classes.

There&#8217;s one final advantage to using Moq which you can&#8217;t easily do with stubs.  It&#8217;s verifying method invocation.  In our sample project, we have a call to _shipmentService.Ship(shipment).  What does Ship() do? It doesn&#8217;t return anything so checking a return parameter is difficult.  It doesn&#8217;t manipulate the shipment (or, let&#8217;s pretend it doesn&#8217;t).  We want to ensure this function is called.  The solution? Use Moq&#8217;s Verify() method, like so:

<pre class="syntax c#">mockShipmentService.Verify(svc => svc.Ship(shipment));
</pre>

This code will throw an exception if Ship wasn&#8217;t called with the shipment variable, ensuring that Ship was actually called if it should.  Using Verify() is helpful when testing out methods returning void- like sending an e-mail, doing a file operation, or ftp.

### Conclusion

Spending some time getting up to speed with a mocking framework will be instramental in learning TDD.  It&#8217;ll also come in handy with just writing unit tests.  Mocking provides a simple, slimmed down solution that&#8217;s much easier to manage than stub classes.

I like Moq because it&#8217;s quick to express what you want done- but the heavy use of lambdas can be a struggle with getting started.  The [moq quickstart][5] is a great guide to the framework and covers all the bases.  The bottom line is don&#8217;t worry about which mocking framework you choose.  They all have their respective advantages.  Pick one, learn it, use it.  If you don&#8217;t like how it does something, switch to something else down the line which does what you want.

[<img src="http://www.dotnetkicks.com/Services/Images/KickItImageGenerator.ashx?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f02%2fusing-moq-3-to-implement-tests-and-avoid-stubs%2f&#038;bgcolor=0000FF" border="0" alt="kick it on DotNetKicks.com" />][6]

 [1]: http://www.michaelhamrah.com/blog/index.php/2008/12/understand-unit-testing-and-td-getting-better-code-coverage/
 [2]: http://www.michaelhamrah.com/blog/wp-content/uploads/2009/02/unittestexamplewithmoq.zip
 [3]: http://martinfowler.com/articles/mocksArentStubs.html
 [4]: http://code.google.com/p/moq/
 [5]: http://code.google.com/p/moq/wiki/QuickStart
 [6]: http://www.dotnetkicks.com/kick/?url=http%3a%2f%2fwww.michaelhamrah.com%2fblog%2findex.php%2f2009%2f02%2fusing-moq-3-to-implement-tests-and-avoid-stubs%2f
