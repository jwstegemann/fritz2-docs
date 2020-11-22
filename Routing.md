---
layout: default
title: Routing
nav_order: 13
---
# Routing

Writing a Single Page Application (SPA), you might need a way to render a certain view depending on url-parameters. This is called routing. 
fritz2 uses the hash-part of the url which starts with a `#`: `https://my.doma.in/path#hash`. url-parameters (`?`) are usually handled by the server, 
while url-hashes (`#`) are handled by the browser.

fritz2 offers a general mechanism for using client-site routing in your SPA. 
It includes a class called `Router` which registers itself to listen for the specific dom-events of the hash changing. 
Additionally, a `Router` needs a `Route` which is an interface you have to implement to use it:
```kotlin
interface Route<T> {
    val default: T
    fun unmarshal(hash: String): T
    fun marshal(route: T): String
}
```
A `Route` always has a default route to start with. It also needs a way to unmarshal and marshal the url-hash string to a kotlin (data-)class and vice versa.

To make this mechanism easier to use, we already implemented two ways of handling routing in fritz2: 
* `StringRoute` uses the url-hash how it is.
* `MapRoute` marshals and unmarshals the url-hash to `Map<String,String>` where `&` is the separator between the entries.

You can easily create a new instance of `Router` by using the global `router()` function. 
There are currently three of them for each type of your `Route`.
* `router(default: String): Router<String>` which uses the `StringRoute`
* `router(default: Map<String, String>): Router<Map<String, String>>` which uses the `MapRoute`
* `<T> router(default: Route<T>): Router<T>` which uses your custom implementation of `Route` interface

Use the last option to implement your own `Route` on your own data type.

Routing is straightforward:

Using simple `String`s by `StringRoute`:
```kotlin
val router = router("welcome")

render {
  section {
    router.renderElement { site ->
      when(site) {
          "welcome" -> div { +"Welcome" }
          "pageA" -> div { +"Page A" }
          "pageB" -> div { +"Page B" }
          else -> div { +"not found" }
      }
    }
  }
}.mount("target")
```

Using a `Map` of parameters by `MapRoute`:
```kotlin
val router = router(mapOf("page" to "welcome"))

render {
    section {
        router.select("page").renderElement { (name, rest) ->
            when(name) {
                "welcome" -> div { +"Welcome" }
                "pageA" -> div { +"Page A" }
                "pageB" -> div { +"Page B" }
                else -> div { +"not found" }
            }
        }
    }
}.mount("target")
```
A router using a `MapRoute` offers an extra `select` method which extract the values as `Pair` for the given key (here `"page"`) 
and requires a function to map the value. Therefore, it returns a `Pair` of the current value and the complete `Map` to
help you decide what to render.

If you want to use your own special `Route` instead, try this:
```kotlin
class SetRoute(override val default: Set<String>) : Route<Set<String>> {
    val separator = "&"
    override fun unmarshal(hash: String): Set<String> = hash.split(separator).toSet()
    override fun marshal(route: Set<String>): String = route.joinToString(separator)
}

val router = router(SetRoute(setOf("welcome")))

render {
    section {
        router.renderElement { route ->
            when {
                route.contains("welcome") -> div { +"Welcome" }
                route.contains("pageA") -> div { +"Page A" }
                route.contains("pageB") -> div { +"Page B" }
                else -> div { +"not found" }
            }
        }
    }
}.mount("target")
```

If you want to change your current route (i.e. when an event fires), you can do so by calling `navTo`: 
```kotlin
val router = router("welcome")

render {
    button {
        +"Navigate to Page A"
        clicks.map { "pageA" } handledBy router.navTo
    }   
}
// or
router.navTo("pageA")
```

Have a look at our [routing example](https://examples.fritz2.dev/routing/build/distributions/index.html)
to see how it works and to play around with it.
