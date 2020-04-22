---
layout: default
title: Components
nav_order: 3
has_children: true
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

More complex components can be realized by using a class. This allows you to encapsulate state, use internal functions to structure your code, etc.

```kotlin
class MyThirdComponent(val p.Person) {
    private fun renderAddress(a: Address = html {
        ... // render street, city, etc.
    }

    fun render = html {
        ... // render name
        p.addresses.forEach {renderAddress(it).bind()}
    }
}
```

Since stateless components alone are not to exciting, go on and read about fritz2 mechanism to handle state, the [Store](Store.html).