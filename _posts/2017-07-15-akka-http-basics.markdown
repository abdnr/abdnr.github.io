---
layout: post
comments: true
title:  "Akka HTTP basics"
date:   2017-07-15 18:10:56 +0200
categories: jekyll update
---

In this post we will have first look at `Akka HTTP` library.

## SBT setup:

{% highlight scala linenos %}
libraryDependencies ++= {
  val akkaVersion = "10.0.9"
  Seq(
    "com.typesafe.akka" %% "akka-http-core" % akkaVersion,
    "com.typesafe.akka" %% "akka-http" % akkaVersion
  )
}
{% endhighlight %}

## Standard server setup

{% highlight scala linenos %}
object Server extends App with Config {
  implicit val system = ActorSystem("server")
  implicit val materializer = ActorMaterializer()
  implicit val executionContext = system.dispatcher

  println(s"Server started at http://$httpHost:$httpPort")
  Await.result(Http().bindAndHandle(route, httpHost, httpPort), Duration.Inf)
}
{% endhighlight %}

## Defining routes

{% highlight scala linenos %}
val route =
  path("") {
    get {
      complete(
        HttpEntity(ContentTypes.`text/html(UTF-8)`, "<h1>Hello Akka!</h1>")
      )
    }
  } ~
  path("other") {
    get {
      entity(as[Entity]) { req =>
        complete(req.value)
      }
    } ~
    get {
      entity(as[String]) { req =>
        complete("not supported")
      }
    }
  }
{% endhighlight %}

## Defining config

{% highlight json linenos %}
http {
  host: "0.0.0.0"
  port: 9000
}
{% endhighlight %}

{% highlight scala linenos %}
import com.typesafe.config.ConfigFactory

trait Config {
  private val config = ConfigFactory.load()

  private val httpConfig = config.getConfig("http")

  val httpHost: String = httpConfig.getString("host")
  val httpPort: Int = httpConfig.getInt("port")
}
{% endhighlight %}

Passing parameters: `-Dhttp.port=12345`

{% include disqus.html %}
