# Future-based backends

There are several backend implementations which are `scala.concurrent.Future`-based. These backends are **asynchronous**, sending a request is a non-blocking operation and results in a response wrapped in a `Future`. 

Apart from the ones described below, also the [Akka](akka.md) backend is `Future`-based.

```eval_rst
===================================== ================================================ ==========================
Class                                 Supported stream type                            Websocket support
===================================== ================================================ ==========================
``AkkaHttpBackend``                   ``akka.stream.scaladsl.Source[ByteString, Any]`` yes (regular & streaming)
``AsyncHttpClientFutureBackend``      n/a                                              no
``OkHttpFutureBackend``               n/a                                              yes (regular)
``HttpClientFutureBackend`` (Java11+) n/a                                              yes (regular)
===================================== ================================================ ==========================
```

## Using async-http-client

To use, add the following dependency to your project:

```scala
"com.softwaremill.sttp.client3" %% "async-http-client-backend-future" % "3.0.0-RC10"
```

And some imports:

```scala
import sttp.client3._
import sttp.client3.asynchttpclient.future.AsyncHttpClientFutureBackend
```

This backend depends on [async-http-client](https://github.com/AsyncHttpClient/async-http-client) and uses [Netty](http://netty.io) behind the scenes.

Next you'll need to create the backend instance:

```scala
val backend = AsyncHttpClientFutureBackend()
```

or, if you'd like to use custom configuration:

```scala
import org.asynchttpclient.AsyncHttpClientConfig

val config: AsyncHttpClientConfig = ???
val backend = AsyncHttpClientFutureBackend.usingConfig(config)
```

or, if you'd like to use adjust the configuration sttp creates:

```scala
import org.asynchttpclient.DefaultAsyncHttpClientConfig

val sttpOptions: SttpBackendOptions = SttpBackendOptions.Default  
val adjustFunction: DefaultAsyncHttpClientConfig.Builder => DefaultAsyncHttpClientConfig.Builder = ???
val backend = AsyncHttpClientFutureBackend.usingConfigBuilder(adjustFunction, sttpOptions)
```

or, if you'd like to instantiate the AsyncHttpClient yourself:

```scala
import org.asynchttpclient.AsyncHttpClient

val asyncHttpClient: AsyncHttpClient = ???  
val backend = AsyncHttpClientFutureBackend.usingClient(asyncHttpClient)
```

## Using OkHttp

To use, add the following dependency to your project:

```scala
"com.softwaremill.sttp.client3" %% "okhttp-backend" % "3.0.0-RC10"
```

and some imports:

```scala
import sttp.client3.okhttp.OkHttpFutureBackend
import scala.concurrent.ExecutionContext.Implicits.global
```

Create the backend using:

```scala
val backend = OkHttpFutureBackend()
```

or, if you'd like to instantiate the OkHttpClient yourself:

```scala
import okhttp3.OkHttpClient

val asyncHttpClient: OkHttpClient = ???  
val backend = OkHttpFutureBackend.usingClient(asyncHttpClient)
```

This backend depends on [OkHttp](http://square.github.io/okhttp/) and fully supports HTTP/2.

## Using HttpClient (Java 11+)

To use, add the following dependency to your project:

```
"com.softwaremill.sttp.client3" %% "httpclient-backend" % "3.0.0-RC10"
```

and some imports:

```scala
import sttp.client3.httpclient.HttpClientFutureBackend
import scala.concurrent.ExecutionContext.Implicits.global
```

Create the backend using:

```scala
val backend = HttpClientFutureBackend()
```

or, if you'd like to instantiate the HttpClient yourself:

```scala
import java.net.http.HttpClient

val client: HttpClient = ???  
val backend = HttpClientFutureBackend.usingClient(client)
```

This backend is based on the built-in `java.net.http.HttpClient` available from Java 11 onwards, works with all Scala versions. A Dotty build is available as well.

## Streaming

The [Akka backend](akka.md) supports streaming using akka-streams.

Other backends don't support non-blocking [streaming](../requests/streaming.md).

## Websockets

Some of the backends (see above) support regular and streaming [websockets](../websockets.md).
