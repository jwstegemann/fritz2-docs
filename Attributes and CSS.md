---
layout: default
title: Attributes and CSS
nav_order: 3
---
# Attributes and CSS

To create rich html-interfaces you will want to use a variety of attributes. In fritz2 there are several ways to easily achieve this depending on your use case.

You can set static values for each `Tag` for `class` and `id` by using the optional parameters of its factory function:
```kotlin
render {
    div("some-static-css-class") {
        button(id = "someId")
    }
}
```

You can set all other attributes inside the `Tag`'s content by calling a function of the according name. 
Every standard html attribute has two functions. One for a static value and a second one for dynamic data coming from a `Flow`.
When coming from a `Flow`, the attribute's value will be updated in the DOM whenever a new value appears on the `Flow`:
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
If you want to set a `Boolean` value, you can set an optional parameter `trueValue` which will be set as value if the data is `true` 
@TODO give an example of trueValue

To set a value for a custom (data-) attribute, use the `attr()`-function. It works for static and dynamic  (from a `Flow`) values:
```kotlin
render {
    div {
        attr("data-something", "someValue")
        attr("data-something", flowOf("someValue"))
    }
}
```

The `class` of a `Tag` is a special attribute. As seen above, you can set a static base-class using the parameter of the `Tag`'s factory-function.
This can be used to add meaning to your html elements by using semantic CSS classnames. Also, it keeps your code cleaner when using CSS frameworks like Bootstrap etc.

You can add dynamic classes by assigning a `Flow` of strings to the `className`-attribute like with any other attribute. 
The same works for `List<String>`s with the `classList`-attribute.

Additionally, you can build a `Map<String, Boolean>` from your model data that enables and disables single classes dynamically:
```kotlin
render {
    div {
        classMap = toDoStore.data.map { mapOf(
            "completed" to it.completed, // a boolean-attribute in the data-model
            "editing" to it.editing
        )}
    }
}
```
Now you can build what you want in your templates, let's create some [Components](Components.html) with it.