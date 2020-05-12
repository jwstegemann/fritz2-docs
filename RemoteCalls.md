---
layout: default
title: Remote Calls
nav_order: 10
---
# Remote Calls

Using the default fetch-api from the browser can get quite tiresome. For this reason fritz2 offers a small wrapper around it:

At first you create a `RequestTemplate` for your backend:
```kotlin
val sampleApi = remote("https://reqresss.in/api/users")
            .acceptJson()
            .header("Content-Type","application/json")
```
The template offers you some [convenience-methods](https://jwstegemann.github.io/fritz2/dokka/fritz2/io.fritz2.remote/-request-template/) to configure your API-calls, like the `acceptJson()` above, that simply adds the correct header to each request that will be sent using the template.

Sending a request is quite straightforward:
```kotlin
            sampleApi.get(s)
                .onErrorLog()
                .body()
```
`body()` return you the body of the response as a `String` (more is yet to come). `onErrorLog` just logs exceptions to the console. Of course you can/should implement your own exception-handling. 

The same works for posts (and other methods), just give it another parameter for the body to send.

Since remote-calls are asynchronous by nature and you do not want to block in your `Handler`s (where you should implement your logic), you have to somehow inject the call, before the `Handler` is called:

```kotlin
val samplePost = apply {s : String ->
            sampleApi.post(body = """
                {
                    "name": "$s",
                    "job": "programmer"
                }
            """.trimIndent())
                .onErrorLog()
                .body()
        } andThen update
``` 

By using `apply` you create just another handler that chains the asnyc remote call and the update-`Handler` in a way that does not block and schedules your update right when you processed the answer from your backend. You can use this `Handler` like any other to handle `Flow`s of actions:

```kotlin
... html {
    button {
        text("add programmer")
        samplePost <= clicks.map {
            "just a name" // wherever you get this from...
        }
    }
...
}
```
Of course you can use `apply` for any other async process, too, or even just to structure your code. 

In the real world you won't want to create the JSON manually, so use [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization).

You can easily setup your local webpack-server to proxy other services when developing locally in your `build.gradle.kts:

```kotlin
kotlin {
    target {
        browser {
            runTask {
                devServer = KotlinWebpackConfig.DevServer(
                    port = 9000,
                    contentBase = listOf("$projectDir/src/main/web"),
                    proxy = mapOf(
                        "/myService" to "http://localhost:8080"
                    )
                )
            }
        }
    }
}
```

Want more? Keep on reading about [Routing](Routing.html).