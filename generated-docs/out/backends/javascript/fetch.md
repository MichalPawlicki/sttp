# JavaScript (Fetch) backend

A JavaScript backend implemented using the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) and backed via `Future`.

This is the default backend, available in the main jar for JS. To use, add the following dependency to your project:

```
"com.softwaremill.sttp.client3" %%% "core" % "3.0.0-RC10"
```

And create the backend instance:

```scala
val backend = FetchBackend()
```

Timeouts are handled via the new [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) class. As this class only recently appeared in browsers you may need to add a [polyfill](https://www.npmjs.com/package/abortcontroller-polyfill).

As browsers do not allow access to redirect responses, if a request sets `followRedirects` to false then a redirect will cause the response to return an error.

Note that `Fetch` does not pass cookies by default. If your request needs cookies then you will need to pass a `FetchOptions` instance with `credentials` set to either `RequestCredentials.same-origin` or `RequestCredentials.include` depending on your requirements.

## Node.js

Running sttp in a node.js will require downloading modules that implement the various classes and functions used by sttp, usually available in browser. At minima, you will need replacement for `fetch`, `AbortController` and `Headers`. To achieve this, you can either use `npm` directly, or the `scalajs-bundler` sbt plugin if you use sbt :

```
npm install --save node-fetch
npm install --save abortcontroller-polyfill
npm install --save fetch-headers
``` 

You then need to load the modules into your runtime. This can be done in
your main method as such :

```scala
val g = scalajs.js.Dynamic.global.globalThis
g.fetch = g.require("node-fetch")
g.require("abortcontroller-polyfill/dist/polyfill-patch-fetch")
g.Headers = g.require("fetch-headers")
```

## Streaming

Streaming support is provided via `FetchMonixBackend`. Note that streaming support on Firefox is hidden behind a flag, see
[ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) for more information.

To use, add the following dependency to your project:

```
"com.softwaremill.sttp.client3" %%% "monix" % "3.0.0-RC10"
```

An example of streaming a response:

```scala   
import sttp.client3._
import sttp.client3.impl.monix._

import java.nio.ByteBuffer
import monix.eval.Task
import monix.reactive.Observable

val backend = FetchMonixBackend()

val response: Task[Response[Observable[ByteBuffer]]] =
  sttp
    .post(uri"...")
    .response(asStreamUnsafe(MonixStreams))
    .send(backend)
```      

```eval_rst
.. note:: Currently no browsers support passing a stream as the request body. As such, using the ``Fetch`` backend with a streaming request will result in it being converted into an in-memory array before being sent. Response bodies are returned as a "proper" stream.
```
