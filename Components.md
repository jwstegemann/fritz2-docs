---
layout: default
title: Components
nav_order: 4
---
# Components

It is very easy to create a lightweight reusable component with fritz2:

```kotlin
val myComponent = render {
    p {
        text("This is the smallest valid stateless component")
        myStore.data.bind()
    }
}
```

fritz2 does not force you to build your components a certain way. You can use every single Kotlin-language-feature you like to do so.

A parametrized component might look like this:

```kotlin
fun myOtherComponent(p: Person) = render {
    p {
        text("Hello, my name is ${p.name}!")
    }
}
```

This function gives you a `Tag`. Use it for example to map the values of a `Flow` and bind it in another component's render-context:

```kotlin
render {
    div {
        myPersonStore.data.map(::myOtherComponent).bind()
    }   
}
``` 

To create a component that can be called inside render-contexts directly, just give it the appropriate receiver:

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

Since stateless components alone are not too exciting, go on and read about fritz2 mechanism to handle state, the [Store](Store.html).