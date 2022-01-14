---
layout: default
title: Http Calls
nav_order: 110
has_children: true
---
# Http Calls

Using the browser's default fetch-api can get quite tiresome, which is why fritz2 offers a small fluent api wrapper for it:

First you create a `Request` which points to your endpoint url:
```kotlin
val usersApi = http("https://reqresss.in/api/users")
            .acceptJson()
            .contentType("application/json")
```
The remote service offers some [convenience-methods](https://api.fritz2.dev/core/core/dev.fritz2.remote/-request/index.html) to configure 
your API-calls, like the `acceptJson()` above, which simply adds the correct header to each request sent using the template.

Sending a request is pretty straightforward:
```kotlin
val result: String = usersApi.get(s).body()
```
`body()` returns the body of the response as a `String`. Instead, you can also use some following methods to get different results:
* `blob(): Blob`
* `arrayBuffer(): ArrayBuffer`
* `formData(): FormData`
* `json(): Any?`

If your request was not successful (`Response.status != 200`) a `FetchException` will be thrown.

The same works for posts and other methods, just use different parameters for the body to send.

Of course the remote service is primarily designed to use in your `Handler`s within your `Store`s when 
exchanging data with your backend:
```kotlin
val userStore = object : RootStore<String>("") {
    
    val usersApi = http("https://reqresss.in/api").acceptJson().contentType("application/json")

    val addUser = handle<String> { _, s : String ->
        usersApi.body("""
            {
                "name": "$s",
                "job": "programmer"
            }
        """.trimIndent())
        .post("users").body()
    }
}
``` 

You can use this `Handler` like any other to handle `Flow`s of actions:

```kotlin
//... 
render {
    div {
        //...
        button {
            +"add programmer"
            clicks.map {
                "just a name" // wherever you get this from...
            } handledBy userStore.addUser
        }
    }
}

// or
userStore.addUser("just a name")
```

To see a complete example of this, visit our 
[remote example](https://examples.fritz2.dev/remote/build/distributions/index.html).

In the real world, instead of creating the JSON manually, better use 
[kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization).
Get inspired by our [repositories example](https://examples.fritz2.dev/repositories/build/distributions/index.html)
and use our [repositories API](Repositories.html).


You can easily setup your local webpack-server to proxy services (avoid CORS, etc.) when developing locally in your `build.gradle.kts`:
```kotlin
kotlin {
    js(IR) {
        browser {
            runTask {
                devServer = devServer?.copy(
                    port = 9000,
                    proxy = mapOf(
                        "/members" to "http://localhost:8080",
                        "/chat" to mapOf(
                            "target" to "ws://localhost:8080",
                            "ws" to true
                        )
                    )
                )
            }
        }
    }.binaries.executable()
}
```
You can find a working example in the [Ktor Chat](https://github.com/jamowei/fritz2-ktor-chat) project.

Want to do bidirectional communications? Keep on reading about [Websockets](Websockets.html).


## Middleware

You can intercept calls made by the remote api for example to implement cross-cutting concerns like logging, generic error handling, etc.

To write your own `Middleware` just implement the following interface:

```kotlin
interface Middleware {
    suspend fun enrichRequest(request: Request): Request
    suspend fun handleResponse(response: Response): Response
}
```

To make a `Request` use a `Middleware` just call its `use`-method:

```kotlin
val myEndpoint = http("/myAPI").use(someMiddleware)
myEndpoint.get("some/Path").body()
```

`enrichRequest` is called before each `Request` you configured to use this `Middleware`. You can add additional headers, parameters, etc. here. `handleResponse` is called on each `Response`.

To implement a simple logging `Middleware` you could write the following: 

```kotlin
val logging = object : Middleware {
    override suspend fun enrichRequest(request: Request): Request {
        console.log("doing request: $request")
        return request
    }

    override suspend fun handleResponse(response: Response): Response {
        console.log("getting response: $response")
        return response
    }
}

val myAPI = http("/myAPI").use(logging)
```

You can add multiple Middlewares in one row by .use(mw1, mw2, mw3). The enrichRequest functions will be called from left to right (mw1,mw2,mw3), the handleResponse functions back from right to left (mw3, mw2, mw1). You can stop the processing of a Response by Middlewares further down the chain by return response.stopPropagation().

fritz2 also offers some base classes to implement the [authentication](Authentication.html)-process of your SPA: 


