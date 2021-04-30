---
layout: default
title: Http Calls
nav_order: 110
---
# Http Calls

Using the browser's default fetch-api can get quite tiresome, which is why fritz2 offers a small wrapper for it:

First you create a `Request` which points to your endpoint url:
```kotlin
val usersApi = http("https://reqresss.in/api/users")
            .acceptJson()
            .contentType("application/json")
```
The remote service offers some [convenience-methods](https://api.fritz2.dev/core/core/dev.fritz2.remote/-request/index.html) to configure 
your API-calls, like the `acceptJson()` above which simply adds the correct header to each request sent using the template.

Sending a request is pretty straightforward:
```kotlin
val result: String = usersApi.get(s).getBody()
```
`getBody()` returns the body of the response as a `String`. Instead, you can also use some following methods to get different results:
* `getBlob(): Blob`
* `getArrayBuffer(): ArrayBuffer`
* `getFormData(): FormData`
* `getJson(): Any?`

If your request was not successful (`Response.status != 200`) a `FetchException` will be thrown.

The same works for posts and other methods, just use different parameters for the body to send.

Of course the remote service is primarily designed to use in your `Handler`s within your `Store`s when 
exchanging data with your backend:
```kotlin
val userStore = object : RootStore<String>("") {
    
    val usersApi = http("https://reqresss.in/api/users").acceptJson().contentType("application/json")

    val addUser = handle<String> { _, s : String ->
        usersApi.body("""
            {
                "name": "$s",
                "job": "programmer"
            }
        """.trimIndent())
        .post().getBody()
    }
}
``` 

You can use this `Handler` like any other to handle `Flow`s of actions:

```kotlin
... 
render {
    div {
        ...
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


You can easily setup your local webpack-server to proxy other services when developing locally in your `build.gradle.kts`:
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
A working example you can find in the [Ktor Chat](https://github.com/jamowei/fritz2-ktor-chat) project.

Want to do bidirectional communications? Keep on reading about [Websockets](Websockets.html).