---
layout: default
title: Components
nav_order: 4
---
# Components

It is very easy to create a lightweight reusable component with fritz2:

```kotlin
val myComponent = html {
    p {
        text("This is the smallest valid stateless component")
    }
}
``` 

fritz2 does not force you to build your components a certain way. You can use every single Kotlin-language-feature you like to do so.

A parametrized stateless component might look like this:

```kotlin
fun myOtherComponent(p: Person) = html {
    p {
        text("Hello, my name is ${p.name}!")
    }
}
```

This function can be called anywhere and gives you a static `Tag` you can use anywhere and mount to any target element using `mount` on it. To create a component that can be used inside
other html-templates just give it the appropriate receiver:

```kotlin
fun HtmlElements.myOtherComponent(p: Person) {
    p {
        text("Hello, my name is ${p.name}!")
    }
}

val myComponent = html {
    div {
        myOtherComponent(p)
    }
}
```

More complex components can be realized by using a class. This allows you to encapsulate state, use internal functions to structure your code, etc.

```kotlin
class MyThirdComponent(val p.Person) {
    private fun HtmlElements.renderAddress(a: Address) {
        ... // render street, city, etc.
    }

    fun HtmlElement.render {
        ... // render name
        p.addresses.forEach {renderAddress(it).bind()}
    }
}
```

Since stateless components alone are not too exciting, go on and read about fritz2 mechanism to handle state, the [Store](Store.html).