---
layout: default
title: Attributes and CSS
nav_order: 40
---
# Attributes

To create rich html-interfaces you will want to use a variety of attributes. In fritz2 there are several easy ways to 
achieve this depending on your use case.

You can set all html-attributes inside the `Tag`'s content by calling a function of the according name. 
Every standard html attribute has two functions. One sets a static value every time the element is re-rendered, 
the second collects dynamic data coming from a [Flow](Flow.html).
When coming from a `Flow`, the attribute's value will be updated in the DOM whenever a new value appears on the `Flow` 
without having to re-render the whole element:
```kotlin
val flowOfInts = ... // i.e. get it from some store

render {
    input {
        placeholder("some text")
        maxLength(flowOfInts)
        disabled(true)
    }
}
```
If you want to set a `Boolean` value, you can set an optional parameter `trueValue` which will be set as the 
attribute-value if your data is `true`:
```kotlin
val isLow = myStore.data.map { i -> i <= 0 }

render {
    button {
        +"My button"
        attr("data-low", isLow, trueValue = "true")
        // isLow == true  -> <button data-low="true">My button</button>
        // isLow == false -> <button>My button</button>
    }
}
```
This is sometimes needed for CSS-selection or animations.

To set a value for a custom (data-) attribute, use the `attr()`-function. It works for static and dynamic (from a `Flow`) values:
```kotlin
render {
    div {
        attr("data-something", "someValue")
        attr("data-something", flowOf("someValue"))
    }
}
```

Sometimes it is important for an attribute to only appear if some condition is `true`, for example some
 [ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) properties like 
[aria-controls](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-controls) should 
preferably appear only if the dependent element exist. The `attr` functions for `Flows` behave in such a way, that
they only set an attribute if the value is *not* `null`. This behaviour could be used to achieve the desired effect:
```kotlin
val isOpened = storeOf(true)

button {
    +"Toggle" 
    // This mechanism is later explained in "State Management". 
    // Just accept for now this simply toggles the boolean value in the store by each click. 
    clicks.map { !isOpened.current } handledBy isOpened.update
 
    attr("aria-controls", isOpened.data.map { if (it) "disclosure" else null })
    //                                                                  ^^^^
    //      make whole attribute disappear if disclosure-div is not rendered
}

isOpened.data.render { 
    if (it) { 
        div(id = "disclosure") {
        +"I am open!"
        }
    }
}
```

# Working with CSS-Classes

The `class` attribute of a `Tag` for working with CSS style-classes is somewhat special.
You can set the static values of each `Tag` for `class` and `id` by using the optional parameters of its factory function:
```kotlin
render {
    div("some-static-css-class") {
        button(id = "someId")
    }
}
```
Use this one-liner to add styling and meaning to your elements by using semantic CSS class-names. 
Also, it keeps your code clean when using CSS frameworks like Bootstrap etc.

To dynamically change the styling of a rendered element, you can add dynamic classes by assigning a `Flow` of strings 
to the `className`-attribute (like with any other attribute).

```kotlin
val enabled = storeOf(true)

div {
    className(enabled.data.map {
        if(it) "background-color: lightgreen;" 
        else "opacity: 0.5; background-color: lightgrey;"
    })
    +"Some important content"
}
```

The same works for `List<String>`s with the `classList`-attribute.

Additionally, you can build a `Map<String, Boolean>` from your model data that enables and disables single classes dynamically:
```kotlin
render {
    div {
        classMap(toDoStore.data.map { mapOf(
            "completed" to it.completed, // a boolean-attribute in the data-model
            "editing" to it.editing
        )})
    }
}
```

fritz2 also offers a function for setting the inline `style` attribute to your elements:
```kotlin
render {
    p {
        inlineStyle("color: red")
        +"this is red text"
    }
}
```
However, we do not recommend using inline styling -  fritz2 offers integrated styling-functions for dealing with 
CSS directly in your Kotlin code, as well as a DSL for working with a themed responsive style system. 
Have a look at [Styling](Styling.html) for more information.

Now that you can build what you want in your templates, let's create some [Components](Components.html) with it.