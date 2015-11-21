---
title: Adding Http Server-Side Events to Akka-Streams
author: Michael
type: post
date: 2015-01-18
url: /2015/01/adding-http-server-side-events-to-akka-streams/
dsq_thread_id:
  - 3432218914
categories:
  - Programming
tags:
  - akka
  - akka-http
  - akka-streams
  - programming
  - scala
  - spray
  - streams
---
In my last blog post we pushed messages from RabbitMq to the console using Akka-Streams. We used the [reactive-rabbit][1] library to create an Akka-Streams `Source` for our _streams-playground_ queue and mapped the stream to a `println` statement before dropping it into an empty `Sink`. All Akka-Streams need both a `Source` and `Sink` to be runnable; we created a complete stream blueprint to be run later.

Printing to the console is somewhat boring, so let&#8217;s take it up a notch. The excellent [Spray Web Service][2] library is being merged into Akka as [Akka-Http][3]. It&#8217;s essentially Spray built with Akka-Streams in mind. The routing dsl, immutable request/response model, and high-performance http server are all there; think of it as Spray vNext. Check out Mathias Doenitz&#8217;s [excellent slide deck on kaka-http from Scala days][4] to learn more on this evolution of Spray; it also highlights the back-pressure functionality Akka-Streams will give you for Http.

Everyone&#8217;s familiar with the Request/Response model of Http, but to show the power of Akka-Streams we&#8217;ll add Heiko Seeberger&#8217;s [Akka-SSE][5] library which brings Server-Side Events to Akka-Http. [Server-Side Events][6] are a more efficient form of long-polling that&#8217;s a lighter protocol to the bi-directional WebSocket API. It allows the client to easily register a handler which the server can then push events to. Akka-SSE adds an SSE-enabled completion marshaller to Akka-Http so your response can be SSE-aware. Instead of printing messages to the console, we&#8217;ll push those messages to the browser with SSE. This shows one of my favorite features of stream-based programming: we simply connect the specific pipes to create more complex flows, without worrying about the how; the framework handles that for us.

## Changing the Original Example

If you&#8217;re interested in the code, simply [clone the original repo][7] with `git clone https://github.com/mhamrah/streams-playground.git` and then `git checkout adding-sse` to get to this step in the repo.

To modify the original example we&#8217;re going to remove the `println` and `Sink` calls from `RabbitMqConsumer` so we can plug in our enhanced `Source` to the Akka-Http sink.

<pre class="scala">def consume() = {
    Source(connection.consume("streams-playground"))
      .map(_.message.body.utf8String)
  }
</pre>

This is now a partial flow: we build up the original RabbitMq `Source` with our map function to get the message body. Now the &#8220;other end&#8221; of the stream needs to be connected, which we defer until later. This is the essence of stream composition. There are multiple ways we can cut this up: our `map` call could be the only thing in this function, with our `RabbitMq` source defined elsewhere.

## Adding Akka-Http

If you&#8217;re familiar with Spray, Akka-Http won&#8217;t look that much different. We want to create an `Actor` for our http service. There are just a few different traits we extend our original Actor from, and a different way plug our routing functions into the Akka-Streams pipeline.

<pre class="scala">class HttpService
  extends Actor
  with Directives
  with ImplicitFlowMaterializer
  with SseMarshalling {
  // implementation
  // ...
}
</pre>

`Directives` gives us the routing dsl, similar to [spray-routing][8] (the functions are pretty much the same). Because Akka-Http uses Akka-Streams, we need an implicit `FlowMaterializer` in scope to run the stream. `ImplicitFlowMaterializer` provides a default. Finally, the `SseMarshalling` trait from Heiko Seeberger&#8217;s library provides the SSE functionality we want for our app. _If you&#8217;re interested in a robust Akka-Streams sample, Heiko&#8217;s [Reactive-Flows][9] is worth checking out._

##Binding to Http

Within our actor body we&#8217;ll create our http stream by binding a routing function to an http port. This is a little different than Spray; there&#8217;s just some syntactical sugar so we can plug our routing function directly into the http pipeline:

<pre class="scala">//need an ExecutionContext for Futures
    import context.dispatcher

    //There's no receive needed, this is implicit
    //by our routing dsl.
    override def receive: Receive = Actor.emptyBehavior

    //We bind to an interface and create a 
    //Flow with our routing function
    Http()(context.system)
      .bind(Config.interface, Config.port)
      .startHandlingWith(route)

    //Simple composition of basic routes
    private def route: Route = sse ~ assets

    //Defined later
    private def see: Route = ???
    private def assets: Route = ???
</pre>

If we weren&#8217;t using the Routing DSL we&#8217;d need to explicitly handling HttpRequest messages in our receive partial function. But the `startHandlingWith` call will do this for us; like spray-routing it takes in a routing function, and will call the appropriate route handler. New http requests will be pumped into the route handler and completed with the completion function at the end of the route.

##Adding SSE

The last piece of the puzzle is adding a specific route for SSE. We need two pieces for SSE support: first, an implicit function which converts the type produced from our `Source` to an `Sse.Message`; in this case, we need to go from a `String` to an `Sse.Message`. Secondly we need a route where a client can subscribe to the stream of server-side events.

<pre class="scala">//Convert a String (our RabbitMq output) to an SSE Message
 implicit def stringToSseMessage(event: String): Sse.Message = {
      Sse.Message(event, Some("published"))
    }

 //add a route for our sse endpoint.
 private def sse: Route = {
      path("messages") {
        get {
          complete {
            RabbitMqConsumer.consume
          }
        }
      }
    }
</pre>

In order for SSE to work in the browser we need to produce a stream of SSE messages with a specific content-type: `Content-Type: text/event-stream`. That&#8217;s what Akka-SSE provides: the SSE Message case classes and serialization to `text/event-stream`. Our implicit function `stringToSseMessage` allows the Scala types to align so the &#8220;stream pipes&#8221; can be attached together. In our case, we produce a stream of `String`s, our RabbitMq message body. We need to produce a stream of `SSE.Messages` so we add a simple conversion function. When a new client connects, they&#8217;ll attach themselves to the consuming RabbitMq `Source`. Akka-Http lets you natively complete a route with a `Flow`; Akka-Sse simply completes that `Flow` with the proper Http response for SSE.

## Trying It Out

Fire up SBT and run `~reStart`, ensuring you have RabbitMq running and set up a queue named `streams-playground` ([see the README][10]). In your console, try a simple `curl` command:

<pre class="bash">$ curl http://localhost:8080/messages
</pre>

The curl command won&#8217;t return. Start sending messages via the RabbitMq Admin console and you&#8217;ll see the SSE output in action:

<pre class="bash">$ curl localhost:8080/messages
event:published
data:woot!

event:published
data:another message!
</pre>

Close the curl command, and fire up your browser at `http://localhost:8080` you&#8217;ll see a simple web page (served from the `assets` route). Continue sending messages via RabbitMq, and those messages will be added to the dom. Most modern browsers natively support SSE with the `EventSource` object. The following gist creates an event listener on the `'published'` event, which is produced from our `implicit string => sse` function above:



There&#8217;s also handlers for opening the initial sse connection and any errors produced. You could also add more events; our simple conversion only goes from a `String` to one specific SSE of type `published`. You could map a set of case classes&#8211;preferably an algebraic data type&#8211;to a set of events for the client. Most modern browsers support `EventStream`; there&#8217;s no need a for an additional framework or library. The gist above includes a test I copied from the [html5 rocks page on SSE][6].

## A Naive Implementation

If you open up multiple browsers to localhost, or `curl http://localhost:8080/messages` a few times, you&#8217;ll notice that a published message only goes to one client. This is because our initial RabbitMq `Source` only consumes one message from a queue, and passes that down the stream pipeline. That single message will only go to one of the connected clients; there&#8217;s no fanout or broadcasting. You can do that with either RabbitMq or Akka-Streams, try experimenting for yourself!

 [1]: https://github.com/ScalaConsultants/reactive-rabbit
 [2]: spray.io
 [3]: http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-M2/scala/http/index.html
 [4]: http://spray.io/scaladays2014/#/
 [5]: https://github.com/hseeberger/akka-sse
 [6]: http://www.html5rocks.com/en/tutorials/eventsource/basics/
 [7]: https://github.com/mhamrah/streams-playground
 [8]: http://spray.io/documentation/1.2.2/spray-routing/
 [9]: https://github.com/hseeberger/reactive-flows
 [10]: https://github.com/mhamrah/streams-playground/blob/master/README.md
