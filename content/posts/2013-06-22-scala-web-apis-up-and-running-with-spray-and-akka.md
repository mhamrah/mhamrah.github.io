---
title: 'Spray API Development: Getting Started with a Spray Web Service Using JSON'
author: Michael
type: post
date: 2013-06-22
url: /2013/06/scala-web-apis-up-and-running-with-spray-and-akka/
dsq_thread_id:
  - 1422598284
categories:
  - Programming
tags:
  - apis
  - json
  - scala
  - spray
  - spray.io
  - web
---
[Spray][1] is a great library for building http api&#8217;s with Scala. Just like [Play!][2] it&#8217;s built with [Akka][3] and provides numerous low and high level tools for http servers and clients. It puts Akka and Scala&#8217;s asynchronous programming model first for high performance, composable application development.

I wanted to highlight the [spray-routing][4] library which provides a nice DSL for defining web services. The routing library can be used with the standalone [spray-can][5] http server or in any servlet container.

We&#8217;ll highlight a simple entity endpoint, unmarshalling Json data into an object and deferring actual process to another Akka actor. To get started with your own spray-routing project, I created a [giter8][6] template to bootstrap your app:

`$g8 mhamrah/sbt -b spray`

[The documentation][7] is quite good and [the source code is worth browsing][8]. For a richer routing example check out [Spray&#8217;s own routing project][9] which shows off http-streaming and a few other goodies.

## Creating a Server

We are going to create three main structures: An actor which contains our Http Service, a trait which contains our route definition, and a Worker actor that will do the work of the request.

The service actor is launched in your application&#8217;s main method. Here we are using Scala&#8217;s App class to launch our server feeding in values from [typesafe config][10]:

    #!scala
    val service= system.actorOf(Props[SpraySampleActor], "spray-sample-service")
    IO(Http) ! Http.Bind(service, system.settings.config.getString("app.interface"), system.settings.config.getInt("app.port"))
    
    println("Hit any key to exit.")
    val result = readLine()
    system.shutdown()
    

Because Spray is based on Akka, we are just creating a standard actor system and passing our service to [Akka&#8217;s new IO library][11]. This is the high performance foundation for our service built on the spray-can server.

## The Service Actor

Our service actor is pretty lightweight, as the functionality is deferred to our route definition in the HttpService trait. We only need to set the actorRefFactory and call runRoutes from our trait. You could simply set routes directly in this class, but the separation has its benefits, primarily for testing.

    #!scala
    class SpraySampleActor extends Actor with SpraySampleService with SprayActorLogging {
      def actorRefFactory = context
      def receive = runRoute(spraysampleRoute)
    }
    

## The Service Trait &#8211; Spray&#8217;s Routing DSL

[Spray&#8217;s Routing DSL][12] is where Spray really shines. It is similar to Sinatra inspired web frameworks like Scalatra, but relies on composable function elements so requests pass through a series of actions similar to [Unfiltered][13]. The result is an easy to read syntax for routing and the Dont-Repeat-Yourself of composable functions.

To start things off, we&#8217;ll create a simple get/post operation at the /entity path:

    #!scala
    trait SpraySampleService extends HttpService {
      val spraysampleRoute = {
        path("entity") {
          get { 
            complete("list")
          } ~
          post {
            complete("create")
          }
        }
      }
    }
    

The path, get and complete operations are [Directives][14], the building blocks of Spray routing. Directives take the current http request and process a particular action against it. The above snippet doesn&#8217;t much except filter the request on the current path and the http action. The path directive also lets you pull out path elements:

    #!scala
    path ("entity" / Segment) { id =>
        get {
          complete(s"detail ${id}")
        } ~
        post {
          complete(s"update ${id}")
        }
      }
    

There are a number ways to pull out elements from a path. Spray&#8217;s unit tests are the best way to explore the possibilities.

You can use curl to test the service so far:

    #!bash
    curl -v http://localhost:8080/entity
    curl -v http://localhost:8080/entity/1234
    

## Unmarshalling

One of the nice things about Spray&#8217;s DSL is how function composition allows you to build up request handling. In this snippet we use json4s support to unmarshall the http request into a JObject:

    #!scala
    /* We need an implicit formatter to be mixed in to our trait */
    object Json4sProtocol extends Json4sSupport {
      implicit def json4sFormats: Formats = DefaultFormats
    }
    
    trait SpraySampleService extends HttpService {
      import Json4sProtocol._
    
      val spraysampleRoute = {
        path("entity") {
          /* ... */
          post {
            entity(as[JObject]) { someObject =>
              doCreate(someObject)
            }
          } 
         /* ... */
      }
    }
    

We use the Entity to directive to unmarshall the request, which finds the implicit json4s serializer we specified earlier. SomeObject is set to the JObject produced, which is passed to our yet-to-be-built doCreate method. If Spray can&#8217;t unmarshall the entity an error is returned to the client.

Here&#8217;s a curl command that sets the http method to POST and applies the appropriate header and json body:

    #!bash
    curl -v -X POST http://localhost:8080/entity -H "Content-Type: application/json" -d "{ \"property\" : \"value\" }"
    

## Leveraging Akka and Futures

We want to keep our route structure clean, so we defer actual work to another Akka worker. Because Spray is built with Akka this is pretty seamless. We need to create our ActorRef to send a message. We&#8217;ll also implement our doCreate function called within the earlier POST /entity directive:

    #!scala
    //Our worker Actor handles the work of the request.
    val worker = actorRefFactory.actorOf(Props[WorkerActor], "worker")
    
    def doCreate[T](json: JObject) = {
      //all logic must be in the complete directive
      //otherwise it will be run only once on launch
      complete {
        //We use the Ask pattern to return
        //a future from our worker Actor,
        //which then gets passed to the complete
        //directive to finish the request.
        (worker ? Create(json))
                    .mapTo[Ok]
                    .map(result => s"I got a response: ${result}")
                    .recover { case _ => "error" }
      }
    }
    

There&#8217;s a couple of things going on here. Our worker class is looking for a Create message, which we send to the actor with the ask (?) pattern. The ask pattern lets us know the task completed so we call then tell the client. When we get the Ok message we simply return the result; in the case of an error we return a short message. The response future returned is passed to Spray&#8217;s complete directive, which will then complete the request to the client. There&#8217;s no blocking occurring in this snippet: we are just wiring up futures and functions.

Our worker doesn&#8217;t do much but out the message contents and return a random number:

    #!scala
    class WorkerActor extends Actor with ActorLogging {
    import WorkerActor._
    
    def receive = {
      case Create(json) => {
        log.info(s"Create ${json}")
        sender ! Ok(util.Random.nextInt(10000))
        }
      }
    }
    

You can view how the entire request is handled [by viewing the source file][15].

## Wrapping Up

Reading the documentation and exploring the unit tests are the best way to understand the power of Spray&#8217;s routing DSL. The performance of the standalone [spray-can][16] service is outstanding, and the Akka platform adds resiliency through its lifecycle management tools. Akka&#8217;s remoting feature allows systems to build out their app tiers. A project I&#8217;m working on is using Spray and Akka to publish messages to a pub/sub system for downstream request handling. It&#8217;s an excellent platform for high performance API development. [Full spray-sample is on GitHub][17].

 [1]: spray.io
 [2]: playframework.com
 [3]: akka.io
 [4]: http://spray.io/documentation/1.1-M8/spray-routing/
 [5]: http://spray.io/documentation/1.1-M8/spray-can/#spray-can
 [6]: https://github.com/n8han/giter8
 [7]: http://spray.io/documentation/
 [8]: https://github.com/spray/spray
 [9]: https://github.com/spray/spray/tree/release/1.1/examples/spray-routing/on-spray-can
 [10]: https://github.com/typesafehub/config
 [11]: http://doc.akka.io/docs/akka/snapshot/scala/io.html
 [12]: http://spray.io/documentation/1.1-M8/spray-routing/key-concepts/routes/
 [13]: http://unfiltered.databinder.net/
 [14]: http://spray.io/documentation/1.1-M8/spray-routing/key-concepts/directives/#directives
 [15]: https://github.com/mhamrah/spray-sample/blob/master/src/main/scala/Boot.scala
 [16]: http://spray.io/documentation/1.1-M8/spray-can/
 [17]: https://github.com/mhamrah/spray-sample
