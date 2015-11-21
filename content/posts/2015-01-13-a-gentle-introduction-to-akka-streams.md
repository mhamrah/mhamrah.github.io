---
title: A Gentle Introduction To Akka Streams
author: Michael
type: post
date: 2015-01-13
url: /2015/01/a-gentle-introduction-to-akka-streams/
dsq_thread_id:
  - 3417007165
categories:
  - Programming
tags:
  - akka
  - akka-streams
  - architecture
  - development
  - programming
  - scala
  - streams
---
I&#8217;m happy to see stream-based programming emerge as a paradigm in many languages. Streams have been around for a while: take a look at the good &#8216;ol | operator in Unix. Streams offer an interesting conceptual model to processing pipelines that is very functional: you have an input, you produce an output. You string these little functions together to build bigger, more complex pipelines. Most of the time you can make these functions asynchronous and parallelize them over input data to maximize throughput and scale. With a Stream, handling data is almost hidden behind the scenes: it just _flows_ through _functions_, producing a new output from some input. In the case of an Http server, the Request-Response model across all clients is a Stream-based process: You map a Request to a Response, passing it through various functions which act on an input. Forget about MVC, it&#8217;s all middleware. No need to set variables, iterate over collections, orchestrate function calls. Just concatenate stream-enabled functions together, and run your code. Streams offer a succinct programming model for a process. The fact it also scales is a nice bonus.

Stream based programming is possible in a variety of languages, and I encourage you to explore this space. There&#8217;s an excellent [stream handbook for Node][1], [an exploratory stream language from Yukihiro &#8220;Matz&#8221; Matsumoto of Ruby fame][2], [Spark Streaming][3] and of course [Akka-Streams][4] which joins the existing [scalaz-stream][5] library for Scala. Even Go&#8217;s [HttpHandler function][6] is Stream-esque: you can easily wrap one function around another, building up a flow, and manipulate the Response stream accordingly.

## Why Akka-Streams?

Akka-Streams provide a higher-level abstraction over Akka&#8217;s existing actor model. The Actor model provides an excellent primitive for writing concurrent, scalable software, but it still is a primitive; it&#8217;s not hard to find a few critiques of the model. So is it possible to have your cake and eat it too? Can we abstract the functionality we want to achieve with Actors into a set of function calls? Can we treat Actor Messages as Inputs and Outputs to Functions, with type safety? Hello, Akka-Streams.

There&#8217;s an excellent [activator template for Akka-Streams][7] offering an in-depth tutorial on several aspects of Akka-Streams. For a more a gentler introduction, read on.

## The Recipe

To cook up a reasonable dish, we are going to consume messages from [RabbitMq][8] with the [reactive-rabbit][9] library and output them to the console. The code is on [GitHub][10]. If you&#8217;d like to follow along, `git clone` and then `git checkout intro`; hopefully I&#8217;ll build up more functionality in later posts so the master branch may differ.

Let&#8217;s start with a code snippet:

<pre class="lang:scala">object RabbitMqConsumer {
 def consume(implicit flowMaterializer: FlowMaterializer) = {
    Source(connection.consume("streams-playground"))
      .map(_.message.body.utf8String)
      .foreach(println(_))
  }
}
</pre>

  * We use a RabbitMq connection to consume messages off of a queue named `streams-playground`.
  * For each message, we pull out the message and decode the bytes as a UTF-8 string
  * We print it to the console

## The Ingredients

  * A `Source` is something which produces exactly one output. If you need something that generates data, you need a `Source`. Our source above is produced from the `connection.consume` function.
  * A `Sink` is something with exactly one input. A `Sink` is the final stage of a Stream process. The `.foreach` call is a Sink which writes the input (_) to the console via `println`.
  * A `Flow` is something with exactly one input and one output. It allows data to flow through a function: like calling `map` which also returns an element on a collection. The `map` call above is a `Flow`: it consumes a `Delivery` message and outputs a `String`.

In order to actually run something using Akka-Streams you must have both a `Source` and `Sink` attached to the same pipeline. This allows you to create a `RunnableFlow` and begin processing the stream. Just as you can compose functions and classes, you can compose streams to build up richer functionality. It&#8217;s a powerful abstraction allowing you to build your processing logic independently of its execution. Think of stream libraries where you &#8220;plug in&#8221; parts of streams together and customize accordingly.

## A Simple Flow

You&#8217;ll notice the above snippet requires an `implicit flowMaterializer: FlowMaterializer`. A `FlowMaterializer` is required to actually run a `Flow`. In the snippet above `foreach` acts as both a `Sink` and a `run()` call to run the flow. If you look at the Main.scala file you&#8217;ll see I start the stream easily in one call:

<pre class="lang:scala">implicit val flowMaterializer = FlowMaterializer()
  RabbitMqConsumer.consume
</pre>

Create a queue named `streams-playground` via the RabbitMq Admin UI and run the application. You can use publish messages in the RabbitMq Admin UI and they will appear in the console. Try some UTF-8 characters, like åßç∂!

## A Variation

The original snippet is nice, but it does require the implicit FlowMaterializer to build and run the stream in `consume`. If you remove it, you&#8217;ll get a compile error. Is there a way to separate the definition of the stream with the running of the stream? Yes, by simply removing the `foreach` call. `foreach` is just syntactical sugar for a `map` with a `run()` call. By explicitly setting a `Sink` without a call to `run()` we can construct our stream blueprint producing a new object of type `RunnableFlow`. Intuitively, it&#8217;s a `Flow` which can be `run()`.

Here&#8217;s the variation:

<pre class="lang:scala">def consume() = {
     Source(connection.consume("streams-playground"))
      .map(_.message.body.utf8String)
      .map(println(_))
      .to(Sink.ignore) //won't start consuming until run() is called!
  }
</pre>

We got rid of our `flowMaterializer` implicit by terminating our Stream with a `to()` call and a simple Sink.ignore which discards messages. This stream will not be run when called. Instead we must call it explicitly in Main.scala:

<pre class="lang:scala">implicit val flowMaterializer = FlowMaterializer()
  RabbitMqConsumer.consume().run()
</pre>

We&#8217;ve separated out the entire pipeline into two stages: the build stage, via the `consume` call, and the run stage, with `run()`. Ideally you&#8217;d want to compose your stream processing as you wire up the app, with each component, like RabbitMqConsumer, providing part of the overall stream process.

## A Counter Example

As an alternative, explore the [rabbitmq tutorials][11] for Java examples. Here&#8217;s a snippet from the site:

<pre class="lang:java">QueueingConsumer consumer = new QueueingConsumer(channel);
    channel.basicConsume(QUEUE_NAME, true, consumer);

    while (true) {
      QueueingConsumer.Delivery delivery = consumer.nextDelivery();
      String message = new String(delivery.getBody());
      System.out.println(" [x] Received '" + message + "'");
    }
</pre>

This is typical of an imperative style. Our flow is controlled by the while loop, we have to explicitly manage variables, and there&#8217;s no flow control. We could separate out the body from the while loop, but we&#8217;d have a crazy function signature. Alternatively on the Akka side there&#8217;s the solid [amqp-client library][12] which provides an Actor based model over RabbitMq:

<pre class="lang:scala">// create an actor that will receive AMQP deliveries
  val listener = system.actorOf(Props(new Actor {
    def receive = {
      case Delivery(consumerTag, envelope, properties, body) => {
        println("got a message: " + new String(body))
        sender ! Ack(envelope.getDeliveryTag)
      }
    }
  }))

  // create a consumer that will route incoming AMQP messages to our listener
  // it starts with an empty list of queues to consume from
  val consumer = ConnectionOwner.createChildActor(conn, Consumer.props(listener, channelParams = None, autoack = false))
</pre>

You get the concurrency primitives via configuration over the actor system, but we still enter imperative-programming land in the Actor&#8217;s `receive` blog (sure, this can be refactored to some degree). In general, if we can model our process as a set of streams, we achieve the same benefits we get with functional programming: clear composition on what is happening, not how it&#8217;s doing it.

Streams can be applied in a variety of contexts. I&#8217;m happy to see the amazing and powerful [spray.io][13] library for Restful web services will be merged into Akka as a stream enabled http toolkit. It&#8217;s also not hard to find out what&#8217;s been done with [scalaz-streams][14] or the plethora of tooling already available in other languages.

 [1]: https://github.com/substack/stream-handbook
 [2]: https://github.com/matz/streem
 [3]: https://spark.apache.org/streaming/
 [4]: http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-M2/index.html
 [5]: https://github.com/scalaz/scalaz-stream
 [6]: http://golang.org/pkg/net/http/#HandleFunc
 [7]: http://www.typesafe.com/activator/template/akka-stream-scala
 [8]: https://www.rabbitmq.com
 [9]: https://github.com/ScalaConsultants/reactive-rabbit
 [10]: https://github.com/mhamrah/streams-playground
 [11]: http://www.rabbitmq.com/tutorials/tutorial-one-java.html
 [12]: https://github.com/sstone/amqp-client
 [13]: http://spray.io
 [14]: https://github.com/scalaz/scalaz-stream#projects-using-scalaz-stream
