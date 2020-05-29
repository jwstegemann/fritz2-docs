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
val s = RootStore<String>("initial value")
```

Every `Store` offers a `Flow` named `data` which can be bound as part of your html:

```kotlin
render {
    p {
        text("actual state = ")
        s.data.bind()
    }
}
```

By calling `s.data.bind()` a [MountPoint](MountPoint.html) is added to the end of your data-flow. This means a DOM-element is created at the mount point (in this example it's a simple `TextNode`) and bound to your `data` so that it will change whenever your `Store`'s state updates. This is called _precise data binding_.

You can of course use every intermediate action like `map`,`filter`, etc., on the `data`-flow as on every other `Flow`:

```kotlin
render {
    p {
        text("you have entered ")
        s.data.map { it.length }.bind()
        text(" characters so far.")
    }
}
```

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
In this case, only the attribute value will change when the model in your store changes. fritz2 offers [pre-defined properties (at each Tag)](https://api.fritz2.dev/fritz2/io.fritz2.dom.html/) for every HTML5-attribute.

But how can you change the model-data in a store? Let's have a look at [State Management](StateManagement.html).