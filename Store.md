---
layout: default
title: Store
nav_order: 5
has_children: true
---
# Store

In fritz2, `Store`s are used to handle your app's state. It heavily depends on Kotlin's `Flow` - if you are not familiar with this concept, please take a look at [Flows](Flows.html) first.

Let's assume the state of your app is a simple `String`. Creating a `Store` to manage that state is quite easy:

```kotlin
val s = object: RootStore<String>("initial value") {
   // space for handlers
}
```

Every `Store` offers a `Flow` named `data` which can be bound as part of your html:

```kotlin
render {
    p {
        s.data.map { "current state = $it" }.bind()
    }
}
```

By calling `s.data.bind()` a [MountPoint](MountPoint.html) is added to the end of your data-flow. This means a DOM-element is created at the mount point (in this example it's a simple `TextNode`) and bound to your `data` so that it will change whenever your `Store`'s state updates. This is called _precise data binding_.

You can of course use every intermediate action like `map`,`filter`, etc., on the `data`-flow as on every other `Flow`:

```kotlin
render {
    p {
        s.data.map { "you have entered ${it.length} characters so far." }.bind()
    }
}
```

To combine data from two or more stores you can use the following `flatMapConcat` -> `map` pattern:
```kotlin
val firstName = object: RootStore<String>("Foo") {}
val lastName = object: RootStore<String>("Bar") {}

render {
    p {
        firstName.data.flatMapConcat { firstName ->
            lastName.data.map { lastName ->
                "Your full name is: $firstName $lastName"
            }
        }.bind()
    }
}.mount("target")
```
Of curse, you can also use a `RootStore<T>` with a complex model which contains all data that you need in one place.
Therefore, take a look in [Nested Structures](NestedStructures.html).

Knowing this, you can easily guess how to derive a reactive component from your state:

```kotlin
val HtmlElements.uppercase
    get() =
        p {
            s.data.map { it.toUpperCase() }.bind()
        }


render {
    div {
        uppercase
    }
}.mount("target")
```

When you want to declare your component inside another function, or you want to pass parameters to it, just use an extension function instead:

```kotlin
fun HtmlElements.uppercaseWithPrefix(prefix: String) =
    p {
        s.data.map { prefix + it.toUpperCase() }.bind()
    }
```

Doing so is called _one-way-databinding_.
You can also bind a `Flow` to an attribute:

```kotlin
form {
    name = store.data.map(...)
}
```
In this case, only the attribute value will change when the model in your store changes. fritz2 offers [pre-defined properties (at each Tag)](https://api.fritz2.dev/core/dev.fritz2.dom.html/) for every HTML5-attribute.

If your store's content is not bound anywhere but need its handler's code to be executed whenever an action is available, you have explicitly `watch()` it:

```kotlin
    val store = object : RootStore<Whatever> {
        val printMessage = handle { model ->
            console.log("some message")
            model
        }
    }

    store.data.watch()
```

But how can you change the model-data in a store? Let's have a look at [State Management](StateManagement.html).