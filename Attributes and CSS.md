---
layout: default
title: Attributes and CSS
nav_order: 3
---
# Attributes and CSS

To create rich HTML-interfaces you will want to use a variety of attributes. In fritz2 there are several ways to easily achieve this depending on your use case.

You can set static values for each `Tag` for `class` and `id` by using the optional parameters of its factory function:
```kotlin
render {
    div("some-static-css-class") {
        button(id = "someId")
    }
}
```

You can set all other attributes inside the `Tag`'s content by assigning a `Flow` of the according type to it. The attribute's value will be updated in the DOM whenever a new value appears on the `Flow`. Wrap a constant with the `const()`-function:
```kotlin
val flowOfInts = ... //i.e. get it from some store

render {
    input {
        placeholder = const("some text")
        maxLength = flowOfInts
    }
}
```

To set a static value for a custom (data-) attribute, use the `attr()`-function:
```kotlin
render {
    div {
        attr("data-something", "someValue")
    }
}
```

You can also bind dynamic values (from a `Flow`) to a custom attribute:
```kotlin
val someFlowOfStrings = ... 

render {
    div {
        someFlowOfStrings.bindAttr("data-something")
    }
}
```

The `class` of a `Tag` is a special attribute. As seen above, you can set a static base-class using the parameter of the `Tag`'s factory-function.

You can add dynamic classes by assigning a `Flow` of strings to the `className`-attribute like with any other attribute. The same works for `List<String>`s with the `classList`-attribute.

Additionally, you can build a map from your model data that enables and disables single classes dynamically:
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
Now that you can build what you want in your templates, let's create some [Components](Components.html) with it.