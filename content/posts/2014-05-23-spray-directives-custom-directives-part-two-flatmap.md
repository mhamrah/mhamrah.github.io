---
title: 'Spray Directives: Custom Directives, Part Two: flatMap'
author: Michael
type: post
date: 2014-05-24
url: /2014/05/spray-directives-custom-directives-part-two-flatmap/
dsq_thread_id:
  - 2708643308
categories:
  - Programming
tags:
  - api
  - Http
  - scala
  - spray
  - web
---
[Our last post covered custom Spray Directives][1]. We&#8217;re going to expand our UUID directive a little further. Generating a unique ID per request is great, but what if we want the client to pass in an existing unique identifier to act as a correlation id between systems?

We&#8217;ll modify our existing directive by checking to see if the client supplied a correlation-id request-header using the existing `optionalHeaderValueByName` directive:

<pre class="syntax scala">def generateUuid: Directive[UUID :: HNil] = {
    optionalHeaderValueByName("correlation-id") {
      case Some(value) => provide(UUID.fromString(value))
      case None => provide(UUID.randomUUID)
    }
  }
</pre>

Unfortunately this code doesn&#8217;t compile! We get an error because Spray is looking for a Route, which is a function of RequestContext => Unit:

<pre class="syntax bash">[error]  found   : spray.routing.Directive1
[error]     (which expands to)  spray.routing.Directive[shapeless.::]
[error]  required: spray.routing.RequestContext => Unit
[error]       case Some(value) => provide(UUID.fromString(value))
</pre>

What do we do? `flatMap` comes to the rescue. Here&#8217;s the deal: we need to transform one directive (`optionalHeaderValueByName`) into another directive (one that provides a UUID). We do this by using flatMap to focus on the value in the first directive (the option returned from `optionalHeaderValueByName`) and return another value (the UUID). With `flatMap` we are basically &#8220;repackaging&#8221; one value into another package.

Here&#8217;s the updated code which properly compiles:

<pre class="syntax scala">def generateUuid: Directive[UUID :: HNil] = {
    //use flatMap to match on the Option returned and provide
    //a new value
    optionalHeaderValueByName("correlation-id").flatMap {
      case Some(value) => provide(UUID.fromString(value))
      case None => provide(UUID.randomUUID)
    }
  }
</pre>

and the test:

<pre class="syntax scala">"can extract a uuid value from the header" in {
      val uuid = java.util.UUID.randomUUID.toString

      Get() ~> addHeader("correlation-id", uuid) ~> uuidRoute ~> check {
        responseAs[String] shouldEqual uuid
      }
    }
</pre>

There&#8217;s a small tweak we&#8217;ll make to our UUID directive to show another example of directive composition. If the client doesn&#8217;t supply a UUID, and we call generateUUID multiple times, we&#8217;ll get different uuids for the same request. This defeats the purpose of a single correlation id, and prevents us from extracting a uuid multiple times per request. A failing test shows the issue:

<pre class="syntax scala">"can extract the same uuid twice per request" in {
      var uuid1: String =""
      var uuid2: String = ""
      Get() ~> generateUuid { uuid =>
        {
          uuid1 = uuid.toString
          generateUuid { another =>
            uuid2 = another.toString
            complete("")
          }
        }
      } ~> check {
        //fails
        uuid1 shouldEqual uuid2
      }
    }
</pre>

To fix the issue, if we generate a UUID, we will add it to the request header as if the client supplied it. We&#8217;ll use the mapRequest directive to add the generated UUID to the header.

<pre class="syntax scala">def generateUuid: Directive[UUID :: HNil] = {
    optionalHeaderValueByName("correlation-id").flatMap {
      case Some(value) => provide(UUID.fromString(value))
      case None =>
        val id = UUID.randomUUID
        mapRequest(r => r.withHeaders(r.headers :+ RawHeader("correlation-id", id.toString))) &#038; provide(id)
    }
  }
</pre>

In my first version I had the mapRequest call and the provide call on separate lines (there was no &). mapRequest was never being called, and it was because mapRequest was not being returned as a value- only the provide directive is returned. We need to &#8220;merge&#8221; these two directives with the & operator. `mapRequest` is a no-op returning a Directive0 (a Directive with a Nil HList) so combining it with provide yields a Directive1[UUID], which is exactly what we want.

 [1]: http://blog.michaelhamrah.com/2014/05/spray-directives-creating-your-own-simple-directive/
