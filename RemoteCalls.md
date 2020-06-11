---
layout: default
title: Remote Calls
nav_order: 10
---
# Remote Calls

Using the browser's default fetch-api can get quite tiresome, which is why fritz2 offers a small wrapper for it:

First you create a `RequestTemplate` for your backend:
```kotlin
val sampleApi = remote("https://reqresss.in/api/users")
            .acceptJson()
            .header("Content-Type","application/json")
```
The template offers some [convenience-methods](https://api.fritz2.dev/core/dev.fritz2.remote/-request-template/) to configure your API-calls, like the `acceptJson()` above which simply adds the correct header to each request sent using the template.

Sending a request is pretty straightforward:
```kotlin
            sampleApi.get(s)
                .onErrorLog()
                .body()
```
`body()` returns the body of the response as a `String` (more is yet to come). `onErrorLog` logs exceptions to the console. Of course, you can and should implement your own exception-handling. 

The same works for posts and other methods, just use different parameters for the body to send.

Since remote-calls are asynchronous by nature, and you do not want to block in your `Handler`s (where you should implement your logic), you have to somehow inject the call before the `Handler` is called:

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

By using `apply`, you create another handler which chains the asnyc remote call and the update-`Handler` in a way that doesn't block. It schedules your update right after you processed the answer from your backend. You can use this `Handler` like any other to handle `Flow`s of actions:

```kotlin
... render {
    button {
        text("add programmer")
        clicks.map {
            "just a name" // wherever you get this from...
        } handledBy samplePost
    }
...
}
```
`apply` can be used for any other async process, or even just to structure your code, too. 

In the real world, instead of creating the JSON manually, better use [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization).

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