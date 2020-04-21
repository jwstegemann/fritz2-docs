---
layout: default
title: Attributes and CSS
nav_order: 3
---
#Attributes and CSS

To create rich HTML-interfaces you have to use many different attributes. In fritz2 there are some different ways to fit every use-case.

On each Tag you can set static values for class and id by just using the optional parameters of it's factory function:

html {
    div("some-static-css-class") {
        button(id = "someId")
    }
}
All the other attributes you can set inside the Tag's content by just just assign it a Flow of the according type. The attribute's value will be updated in the DOM whenever a new value appears on the Flow. If the value is constant, just wrap it with the const()-function:

val flowOfInts = ... //i.e. get it from some Store

html {
    input {
        placeholder = const("some text")
        maxLength = flowOfInts
    }
}
If you want to set a static value for a custom (data-) attribute, use the attr()-function:

html {
    div {
        attr("data-something", "someValue")
    }
}
Of course you can also bind dynamic values (from a Flow) to a custom attribute:

val someFlowOfStrings = ... 

html {
    div {
        someFlowOfStrings.bindAttr("data-something)
    }
}
A very special attribute is the class of a Tag. As seen above, you can set a static base-class just by the parameter of the factory-function of the Tag.

You can add dynamic classes by assign a Flow of Strings to the classMap-attribute as with any other attribute. The same works for List<String>s with the classList-attribute.

In addition to this, you can build a map from you model data, that enables and disables single classes dynamically:

html {
    div {
        classMap = toDoStore.data.map { mapOf(
            "completed" to it.completed, // a boolean-attribute in the data-model
            "editing" to it.editing
        )}
    }
}