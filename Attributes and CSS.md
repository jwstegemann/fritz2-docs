---
layout: default
title: Attributes and CSS
nav_order: 40
---
# Attributes

To create rich html-interfaces you will want to use a variety of attributes. In fritz2 there are several ways to easily 
achieve this depending on your use case.

You can set all html-attributes inside the `Tag`'s content by calling a function of the according name. 
Every standard html attribute has two functions. One to set a static value (every time the element is re-rendered), 
and a second one for dynamic data coming from a `Flow`.
When coming from a `Flow`, the attribute's value will be updated in the DOM whenever a new value appears on the `Flow` 
without having to re-render the whole element:
```kotlin
val flowOfInts = ... //i.e. get it from some store

render {
    input {
        placeholder("some text")
        maxLength(flowOfInts)
        disabled(true)
    }
}
```
If you want to set a `Boolean` value, you can set an optional parameter `trueValue` which will be set as the 
attribute-value if your data is `true` 
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

# Working with CSS-Classes

The `class` attribute of a `Tag` used to work with CSS style-classes is somehow special:

You can set static values for each `Tag` for `class` and `id` by using the optional parameters of its factory function:
```kotlin
render {
    div("some-static-css-class") {
        button(id = "someId")
    }
}
```
This can be used to add styling and meaning to your elements in one row by using semantic CSS class-names. 
Also, it keeps your code cleaner when using CSS frameworks like Bootstrap etc.

To dynamically change the styling of a rendered element, you can add dynamic classes by assigning a `Flow` of strings 
to the `className`-attribute (like with any other attribute). 
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

Beyond that fritz2 offers integrated styling-functions to deal with CSS directly in your Kotlin code as well as a 
DSL for working with a themed responsive style system. Have a look at [Styling](Styling.html) for more information.

Now you can build what you want in your templates, let's create some [Components](Components.html) with it.