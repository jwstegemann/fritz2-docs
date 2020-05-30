---
layout: default
title: Components
nav_order: 4
---
# Components

It's very easy to create a lightweight reusable component with fritz2:

```kotlin
val myComponent = render {
    p {
        text("This is the smallest valid stateless component")
        myStore.data.bind()
    }
}
```

fritz2 doesn't force you to build your components a certain way - you can use every single Kotlin-language-feature you like.
A parameterized component could look like this:

```kotlin
fun myOtherComponent(p: Person) = render {
    p {
        text("Hello, my name is ${p.name}!")
    }
}
```

This function gives you a `Tag` which has multiple purposes, such as mapping the values of a `Flow` and binding it in another component's render-context:

```kotlin
render {
    div {
        myPersonStore.data.map(::myOtherComponent).bind()
    }   
}
``` 

To create a component which can be called inside render-contexts directly, just specify the appropriate receiver:

```kotlin
fun HtmlElements.myOtherComponent(p: Person) {
    p {
        text("Hello, my name is ${p.name}!")
    }
}

val myComponent = render {
    div {
        myOtherComponent(p)
    }
}
```

Since stateless components alone are not that exciting, go on and read about the fritz2 mechanism to handle state: the [Store](Store.html).