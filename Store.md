---
layout: default
title: Store
nav_order: 5
has_children: true
---
# Store

In fritz2, `Store`s are used to handle your app's state. 
It heavily depends on Kotlin's `Flow` - if you are not familiar with this concept, please take a look at [Flows](Flows.html) first.

Let's assume the state of your app is a simple `String`. Creating a `Store` to manage that state is quite easy:

```kotlin
val s = storeOf("initial value")
```

Every `Store` offers a `Flow` named `data` which can be bound as part of your html:

```kotlin
render {
    p {
        s.data.bind()
    }
}.mount("target")
```

By calling `s.data.bind()` a [MountPoint](MountPoint.html) is created and collects your model values. 
This means a DOM-element is created (in this example it's a simple `TextNode`) and 
bound to your `data` so that it will change whenever your `Store`'s state updates. This is called _precise data binding_.

You can of course use every intermediate action like `map`,`filter`, etc., on the `data`-flow as on every other `Flow`:

```kotlin
render {
    p {
        s.data.map { "you have entered ${it.length} characters so far." }.bind()
    }
}.mount("target")
```

To combine data from two or more stores you can use the `combine` method:
```kotlin
val firstName = object : RootStore<String>("Foo") {}
val lastName = object : RootStore<String>("Bar") {}

render {
    p {
        firstName.data.combine(lastName.data) { firstName, lastName ->
            "Your full name is: $firstName $lastName"
        }.bind()
    }
}.mount("target")
```
Of course, you can also use a `RootStore<T>` with a complex model which contains all data that you need in one place.
Therefore, take a look at [Nested Structures](NestedStructures.html).

You can also bind a `Flow` to an attribute:
```kotlin
input {
    value = store.data
}
```
In this case, only the attribute value will change when the model in your store changes. 
fritz2 offers [pre-defined properties (at each Tag)](https://api.fritz2.dev/core/dev.fritz2.dom.html/) for every HTML5-attribute.

But how can you change the model-data in a store? Let's have a look at [State Management](StateManagement.html).