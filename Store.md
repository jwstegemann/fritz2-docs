---
layout: default
title: Store
nav_order: 4
has_children: true
---
# Store

In fritz2 you will use one or more `Store`s to handle you app's state. It heavily depends on Kotlin's `Flow`. If you are not familiar with this concept, please read a little about [[Flows]] first.

Let's assume, the state of your app is a simple `String`. Creating a `Store` to manage that state is quite easy:

```kotlin
val s = RootStore<String>("initial value")
```

Every `Store` offers a `Flow` name `data`, that you can be bound to an element in your html:

```kotlin
html {
    p {
        text("actual state = ")
        s.bind()   
    }
}
```

By calling `s.bind()` you create a [[MountPoint]] as the end of your data-flow. That means that a DOM-element is created by the mount point (in our case a simple `TextNode`) and bound to your `data` so that it will change whenever that state in your `Store` is updated. This is what is called _precise data binding_.

Of course you can call every intermediate action on the `data`-flow as on every other `Flow`, like `map`,`filter`, etc.:

```kotlin
html {
    p {
        text("you have entered ")
        s.map { it.length }.bind()
        text(" characters so far.")
}
```

Knowing this, you can easily assume, how you can derive a reactive component from your state:

```kotlin
val uppercase = data.map {
    html {
        p {
            it.toUpperCase().bind()
        }
    } 
}

html {
    div {
        uppercase.bind()
    }
}.mount("target")
```

Doing so is called _one-way-databinding_.
You can also bind a Flow to an attribute:

```kotlin
div {
    class = store.data.map(...)
}
```
In this case just the attribute value will change when the model in your store changes. fritz2 offers [pre-defined properties](https://jwstegemann.github.io/fritz2/dokka/fritz2/io.fritz2.dom.html/) for every HTML5-attribute.

But how can you change the model-data in a store? Let's have a look at [State Management](StateManagement.html).