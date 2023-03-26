---
title: Building a Reactive API Architecture with Akka and Scala
created: 2023-03-26-10:44:46
---

# Building a Reactive API Architecture with Akka and Scala

Modern software development demands that applications are built to be responsive, resilient, and elastic. As a result, building a reactive API architecture has become a popular approach, and Akka and Scala are commonly used to achieve this. In this blog post, we'll explore how to build a reactive API using Akka and Scala.

## What is a reactive API?

Before we dive into the details of building a reactive API with Akka and Scala, it's important to understand what a reactive API is. At its core, a reactive API is designed to be highly responsive and provide a great user experience. It also ensures that your application can scale and remain resilient under high load conditions.

A reactive API architecture is event-driven, meaning that the events trigger the responses. The underlying idea is that events trigger a system reaction, and this reaction should be proportional to the event. The system should not only react, but it should also be reliable and recoverable.

## What is Akka?

Akka is a toolkit and runtime that allows developers to build highly concurrent, distributed, and fault-tolerant event-driven systems. Akka is written in Scala but can be used with other programming languages like Java and Python.

The toolkit is designed around the actor model of concurrency. An actor is an independent unit of computation that communicates asynchronously with other actors, with each actor processing messages in a specific order. In Akka, actors form the core building blocks of the system.

## Building a Reactive API with Akka and Scala

Now that we've covered the basics of reactive API and Akka let's see how we can combine the two to build a reactive API with Scala.

### Define API Endpoints

Before we start working with Akka, we need to define the endpoints of our API. We can use Akka HTTP to define our API endpoints, like this:

```scala
import akka.http.scaladsl.server.Route
import akka.http.scaladsl.server.Directives._

class ApiRoutes() {
  def routes: Route = 
    path("hello") {
      get {
        complete("Hello, world!")
      }
    }
}
```

This code defines a simple API endpoint at the "/hello" path, which returns "Hello, world!" when the GET method is called.

### Define an Actor System

We'll now define our actor system to start processing incoming requests. We'll create actors using Akka to handle all incoming requests. We'll also define actors for incoming WebSocket messages and outgoing WebSockets to communicate with clients.

```scala
import akka.actor.ActorSystem

object Main {
  def main(args: Array[String]) {
    implicit val system = ActorSystem("MyActorSystem")
  }
}
```

### Create an Actor

Next, we'll create an actor to handle incoming messages. We'll define an actor that will convert the incoming request messages to JSON and pass the JSON to the desired endpoint.

```scala
import akka.actor.Actor
import spray.json._
import akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport
import scala.concurrent.duration._

class MyActor extends Actor with SprayJsonSupport with DefaultJsonProtocol {
  import context.dispatcher

  context.system.scheduler.schedule(1.second, 5.seconds, self, "tick")

  var client: Option[ActorRef] = None

  def receive = {
    case "tick" =>
      client.foreach(_ ! (JObject(Fields.Empty), "time!" -> JsString(DateTime.now().toString)))
    case x =>
      if (client.isDefined) client.foreach(_ ! x)
  }
}
```

### Create a WebSocket Flow

We'll then define a WebSocket flow using Akka HTTP. The WebSocket flow will handle the WebSocket traffic in our application. The WebSocket flow will send messages to the WebSocket actors, which will be translated to JSON and sent to the desired endpoint.

```scala
import akka.http.scaladsl.model.ws.{Message, TextMessage}
import akka.stream.scaladsl.{Flow, Sink, Source}
import akka.actor.{Actor, ActorRef, Props}
import scala.concurrent.duration._

class MyWebSocketActor extends Actor with SprayJsonSupport with DefaultJsonProtocol {
  import context.dispatcher

  var client: Option[ActorRef] = None

  def receive = {
    case _: HttpUpgrade =>
      client = Some(sender())
      context.system.scheduler.schedule(1.second, 5.seconds, self, "tick")
    case TextMessage.Strict(text) =>
      println("Received WebSocket message: " + text)
    case x =>
      if (client.isDefined) client.foreach(_ ! x)
  }
}

trait ChatResource {
  import akka.stream.ActorMaterializer
  import system.dispatcher

  implicit val materializer: ActorMaterializer = ActorMaterializer()
  implicit val timeout: Timeout = 30.seconds

  class ChatRoutes() {
    def routes: Route =
      path("chat") {
        parameter('name) { name =>
          get {
            handleWebSocketMessages(chatroomFlow(name))
          }
        }
      }

    def chatroomFlow(name: String): Flow[Message, Message, Unit] = {
      val chat = system.actorOf(Props[MyActor])

      val incomingMessages: Sink[Message, NotUsed] =
        Flow[Message].map {
          case TextMessage.Strict(text) =>
            IncomingMessage(text, name)
        }.to(Sink.actorRef[IncomingMessage](chat, ClientLeft(name)))

      val outgoingMessages: Source[Message, NotUsed] =
        Source.actorRef[OutgoingMessage](10, OverflowStrategy.dropTail)
          .mapMaterializedValue(chat ! IncomingConnection(name, _))

      Flow.fromSinkAndSource(incomingMessages, outgoingMessages)
        .wireTap(_.watchTermination() {
          case (_, done) ⇒
            done.onComplete(_ => chat ! ClientLeft(name))
        })
    }
  }
}
```

This code snippet defines an actor that handles WebSocket traffic and a WebSocket flow that listens to incoming traffic and sends traffic to our actors.

### Define Error Handlers

We'll also define error handlers in case something goes wrong during API processing. These error handlers can be used to produce user-friendly error messages, which can help developers effectively debug issues.

```scala
import akka.http.scaladsl.{Http, HttpsConnectionContext}
import akka.stream.ActorMaterializer

class WebServer(apiRoutes: ApiRoutes, chatResource: ChatResource) {
  implicit val system: ActorSystem = ActorSystem("my-system")
  implicit val materializer: ActorMaterializer = ActorMaterializer()
  implicit val executionContext: ExecutionContextExecutor = system.dispatcher

  def run(): Unit = {
    val http = Http(system)

    val bindingFuture = http.bindAndHandle(
      apiRoutes.routes ~ chatResource.routes,
      "localhost",
      8080)

    println(s"Server online at http://localhost:8080/")

    Await.result(system.whenTerminated, Duration.Inf)
  }
}

object WebServer {
  def main(args: Array[String]): Unit = {
    val apiRoutes = new ApiRoutes()
    val chatResource = new ChatResource()

    new WebServer(apiRoutes, chatResource).run()
  }
}
```

This code defines a WebServer class which initializes and starts our server. We could define error handlers in this class to catch and handle errors in a user-friendly manner.

## Conclusion

In this blog post, we explored how to build a reactive API with Akka and Scala. We showed how to define API endpoints and how to create actors to handle incoming messages. We also defined a WebSocket flow to handle WebSocket traffic and error handlers to produce user-friendly error messages.

We've only scratched the surface of what is possible with Akka and Scala. By using these tools, developers can build highly concurrent, distributed, and fault-tolerant event-driven systems. The actor model provides a great level of scalability and can handle many requests simultaneously, all while remaining highly responsive.

Reactive API architecture is becoming more and more common in modern software development, and Akka and Scala are perfect for achieving it. With the tips and tricks shown in this blog post, you can now start your own journey into building reactive API architectures with Akka and Scala.
