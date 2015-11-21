---
title: 'Spray Directives: Creating Your Own, Simple Directive'
author: Michael
type: post
date: 2014-05-24
url: /2014/05/spray-directives-creating-your-own-simple-directive/
dsq_thread_id:
  - 2708542702
categories:
  - Programming
tags:
  - api
  - Http
  - scala
  - spray
  - web
---
The [spray-routing][1] package provides an excellent dsl for creating restful api&#8217;s with Scala and Akka. This dsl is powered by [directives][2], small building blocks you compose to filter, process and compose requests and responses for your API. Building your own directives lets you create reusable components for your application and better organize your application.

I recently refactored some code in a Spray API to leverage custom directives. The [Spray documentation provides a good reference on custom directives][3] but I found myself getting hung up in a few places.

As an example we&#8217;re going to write a custom directive which produces a UUID for each request. Here&#8217;s how I want to use this custom directive:

<pre class="syntax scala">generateUUID { uuid =>
  path("foo") {
   get {
     //log the uuid, pass it to your app, or maybe just return it
     complete { uuid.toString }
   }
  }
}
</pre>

Usually you leverage existing directives to build custom directives. I (incorrectly) started with the `provide` directive to provide a value to an inner route:

<pre class="syntax scala">import spray.routing._
import java.util.UUID
import Directives._

trait UuidDirectives {
  def generateUuid: Directive1[UUID] = {
    provide(UUID.randomUUID)
  }
}
</pre>

Before I explain what&#8217;s wrong, let&#8217;s dig into the code. First, generateUuid is a function which returns a Directive1 wrapping a UUID value. Directive1 is just a type alias for `Directive[UUID :: HNil]`. Directives are centered around a feature of the shapeless library called heterogeneous lists, or HLists. An `HList` is simply a list, but each element in the list can be a different, specific type. Instead of a generic `List[Any]`, your list can be composed of specific types of list of String, Int, String, UUID. The first element of this list is a String, not an Any, and the second is an Int, with all the features of an Int. In the directive above I just have an `HList` with one element: `UUID`. If I write `Directive[UUID :: String :: HNil]` I have a two element list of `UUID` and String, and the compiler will throw an error if I try to use this directive with anything other a `UUID` and a String. HLists sound like a lightweight case class, but with an `HList`, you get a lot of list-like features. HLists allow the compiler to do the heavy lifting of type safety, so you can have strongly-typed functions to compose together.

Provide is a directive which (surprise surprise) will provide a value to an inner route. I thought this would be perfect for my directive, and the corresponding test ensures it works:

<pre class="syntax scala">import org.scalatest._
import org.scalatest.matchers._
import spray.testkit.ScalatestRouteTest
import spray.http._
import spray.routing.Directives._

class UuidDirectivesSpec
  extends FreeSpec
  with Matchers
  with UuidDirectives
  with ScalatestRouteTest {

  "The UUID Directive" - {
    "can generate a UUID" in {
      Get() ~> generateUuid { uuid => complete(uuid.toString) } ~> check  {
        responseAs[String].size shouldBe 36
      }
    }
  }
}
</pre>

But there&#8217;s an issue! Spray directives are classes are composed when instantiated via an apply() function. The [Spray docs on understanding the dsl structure][4] explains it best, but in summary, generateUuid will only be called once when the routing tree is built, not on every request.

A better unit test shows the issue:

<pre class="syntax scala">"will generate different UUID per request" in {
      //like the runtime, instantiate route once
      val uuidRoute =  generateUuid { uuid => complete(uuid.toString) }

      var uuid1: String = ""
      var uuid2: String = ""
      Get() ~> uuidRoute ~> check  {
        responseAs[String].size shouldBe 36
        uuid1 = responseAs[String]
      }
      Get() ~> uuidRoute ~> check  {
        responseAs[String].size shouldBe 36
        uuid2 = responseAs[String]
      }
      //fails!
      uuid1 shouldNot equal (uuid2)
    }
  }
</pre>

The fix is simple: we need to use the `extract` directive which applies the current RequestContext to our route so it&#8217;s called on every request. For our UUID directive we don&#8217;t need anything from the request, just the function which is run for every request:

<pre class="syntax scala">trait UuidDirectives {
  def generateUuid: Directive[UUID :: HNil] = {
    extract(ctx =>
        UUID.randomUUID)
  }
}
</pre>

With our randomUUID call wrapped in an extract directive we have a unique call per request, and our tests pass!

In a following post we&#8217;ll add some more complexity to our custom directive, stay tuned!

 [1]: http://spray.io/documentation/1.2.1/spray-routing/
 [2]: http://spray.io/documentation/1.2.1/spray-routing/key-concepts/directives/
 [3]: http://spray.io/documentation/1.2.1/spray-routing/advanced-topics/custom-directives/
 [4]: http://spray.io/documentation/1.2.1/spray-routing/advanced-topics/understanding-dsl-structure/
