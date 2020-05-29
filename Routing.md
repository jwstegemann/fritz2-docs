---
layout: default
title: Routing
nav_order: 11
---
# Routing

Writing a Single Page Application (SPA), you might need a way to render a certain view depending on url-parameters. This is called routing. 
fritz2 uses the hash-part of the url which starts with a `#`: `https://my.doma.in/path#hash`. url-parameters (`?`) are usually handled by the server, while url-hashes (`#`) are handled by the browser.

Fritz2 offers a general mechanism for using client-site routing in your SPA. It includes a class called `Router` which registers itself to listen for the specific dom-events of the hash changing. Additionally, a `Router` needs a `Route` which is an interface you have to implement to use it:
```kotlin
interface Route<T> {
    val default: T
    fun unmarshal(hash: String): T
    fun marshal(route: T): String
}
```
A `Route` always has a default route to start with. It also needs a way to unmarshal and marshal the url-hash string to a kotlin (data-)class and vice versa.

To make this mechanism easier to use, we implemented two ways of handling routing: 
* `StringRoute` uses the url-hash how it is.
* `MapRoute` marshals and unmarshals the url-hash to `Map<String,String>` where `&` is the separator between the entries.

If none of these fits your needs, you can implement your own `Route` on your own data type instead.

The usage of routing is straightforward:

Using simple `String`s by `StringRoute`:
```kotlin
val router = router("welcome")

render {
  section {
    router.routes.map { site ->
      when(site) {
          "welcome" -> div { +"Welcome" }
          "pageA" -> div { +"Page A" }
          "pageB" -> div { +"Page B" }
          else -> div { +"not found" }
      }
    }.bind()
  }
}       
```

Use a `Map` of parameters by `MapRoute`:
```kotlin
val router = router(mapOf("page", "welcome"))

render {
  section {
    router.select("page") {
      val (name, rest) = it
      when(name) {
          "welcome" -> div { +"Welcome" }
          "pageA" -> div { +"Page A" }
          "pageB" -> div { +"Page B" }
          else -> div { +"not found" }
      }
    }.bind()
  }
}       
```
A router using a `MapRoute` offers an extra `select` method which extract the values for the given key (here `"page"`) and needs a function to map the value. Therefore, it returns a `Pair` of the current value and the complete `Map`, to help you decide what to render.

If you want to use your own special `Route` instead, try this:
```kotlin
class SetRoute(override val default: Set<String>) : Route<Set<String>> {
  override fun unmarshal(hash: String): Set<String> {
      TODO("Not yet implemented")
  }

  override fun marshal(route: Set<String>): String {
      TODO("Not yet implemented")
  }
}

val router = router(SetRoute(setOf("welcome")))

render {
  section {
    router.routes.map { set ->
      TODO("Not yet implemented")
    }.bind()
  }
}  
```
These are the options for routing with fritz2.

If you want to change your current route (i.e. when an event fires) you can do this by calling `navTo`: 
```
button {
    text("Navigate to Page A")
    clicks.map { mapOf("page" to "pageA") } handledBy router.navTo
}
```
Also, you can set the url-hash to an initial page request, for example `https://mysite.com/myapp#page=welcome`.
