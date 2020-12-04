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
To create a component which can be called inside render-contexts directly, just specify the appropriate receiver
 `RenderContext`:

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
}.mount("target")

// or with Flows

render {
    div {
        // caution: this re-renders everything when the person inside the Flow is changed
        someflowOfPerson.render { person ->
            div {
                myOtherComponent(person)
            }
        }
    }
}.mount("target")
```

To create more nested components do the following:
```kotlin
// needs to return a html element for using in render{} method directly
fun RenderContext.container(content: Div.() -> Unit): Div {
    return div("container") {
        content()
    }
}

// don't need to return a html element because its used 
// in HtmlElements context only
fun RenderContext.okBtn(content: Button.() -> Unit): Button {
    return button("btn success") {
        +"Ok"
        content()
    }
}

render {
    container {
        p {
            text("Hello World!")
        }
        okBtn {
            clicks handledBy someStore.someHandler   
        }
    }
}.mount("target")
```

Since stateless components alone are not that exciting, go on and read about the fritz2 
mechanism to handle state: the [Store](Store.html).