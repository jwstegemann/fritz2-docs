---
layout: default
title: Remote Calls
nav_order: 10
---
# Remote Calls

Using the browser's default fetch-api can get quite tiresome, which is why fritz2 offers a small wrapper for it:

First you create a `Request` for your backend:
```kotlin
val sampleApi = remote("https://reqresss.in/api/users")
            .acceptJson()
            .header("Content-Type","application/json")
```
The template offers some [convenience-methods](https://api.fritz2.dev/core/dev.fritz2.remote/-request-template/) to configure your API-calls, like the `acceptJson()` above which simply adds the correct header to each request sent using the template.

Sending a request is pretty straightforward:
```kotlin
            sampleApi.get(s)
                .body()
```
`body()` returns the body of the response as a `String` (more is yet to come).

The same works for posts and other methods, just use different parameters for the body to send.

Remote-calls are asynchronous by nature, using fritz2-remote-api, your `Handler` will suspend until the response arives and is read by the browser:

```kotlin
val save = handle<String> { s : String ->
            sampleApi.post(body = """
                {
                    "name": "$s",
                    "job": "programmer"
                }
            """.trimIndent())
                .onErrorLog()
                .body()
        }
``` 
In the real world, instead of creating the JSON manually, better use [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization). 

You can easily setup your local webpack-server to proxy other services when developing locally in your `build.gradle.kts:

```kotlin
kotlin {
    target {
        browser {
            runTask {
                devServer = KotlinWebpackConfig.DevServer(
                    port = 9000,
                    contentBase = listOf("$projectDir/src/jsMain/resources"),
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