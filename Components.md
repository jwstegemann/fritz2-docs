---
layout: default
title: Components
nav_order: 50
---
# Components

It's very easy to create a lightweight reusable component with fritz2. Basically all you have to do is writing a function with `RenderContext` as it's receiver type:

```kotlin
fun RenderContext.myComponent() {
    p {
        +"This is the smallest valid stateless component"
    }
}

render {
    myComponent()
}
```

That way it's also straight forward to parametrize your component:

```kotlin
fun RenderContext.myOtherComponent(person: Person): P {
    return p {
        +"Hello, my name is ${person.name}!"
    }
}

val somePerson = Person(...)
render {
    div {
        myOtherComponent(somePerson)
    }
}
```

To allow nested components use a lamba with `RenderContext` as its receiver or the type of the element you are calling this lambda in:
```kotlin
// return a html element if your if you need it
fun RenderContext.container(content: Div.() -> Unit): Div {
    return div("container") {
        content()
    }
}

render {
    container {
        p {
            text("Hello World!")
        }

        clicks handledBy someHandler // you will see what this does in the next chapter
    }
}
```

Using `Div` as receiver type in the example above allows you to access the attributes and events of your container-element from your content-lambda. Use `RenderContext` if this is not necessary or intended.

Since stateless components alone are not that exciting, go on and read about the fritz2 
mechanism to handle state: the [Store](Store.html).