---
layout: default
title: Lists in a Model
nav_order: 90
---
# Lists as a Model

Like for any other type, you can create a `Store` that holds a list:

```kotlin
val listStore = storeOf(listOf("a","b","c")
```

It is perfectly valid to handle this as seen before: render the `data` by iterating over the `List`:

```kotlin
listStore.data.render { list ->
    list.forEach {
        span { + it }
    }
}
```

Keep in mind that this means re-rendering *all* `span`s in this example the list changes, regardless if you just added a new `String` at the end or changed the `String` at index 2. This might be exactly what you want for small `List`s, 
for `List`s that changes rarely or not at all, or for `List`s with a small representation in HTML (just a text per item), etc.
However, for `List`s that change more often and/or result in complex HTML-trees per item, this does not perform well.

For those cases, fritz2 offers the method `renderEach {}` which creates a `RenderContext` and mounts its result to the DOM. 
`renderEach {}` works analogously to `render {}`, but it creates a specialized mount-point in order to 
identify elements for re-rendering. This mount-point compares the last version of your list with the 
new one on every change and applies the minimum necessary patches to the DOM.

```kotlin
val seq = object : RootStore<List<String>>(listOf("one", "two", "three")) {
    var count = 0

    val addItem = handle { list ->
        count++
        list + "yet another item no. $count"
    }
    val deleteItem = handle<String> { list, current ->
        list.minus(current)
    }
}

render {
    section {
        ul {
            seq.data.renderEach { s ->
                li {
                    button("btn", id = "delete-btn") {
                        +s
                        clicks.map { console.log("deleting $s"); s } handledBy seq.deleteItem
                    }
                }
            }
        }
        button("button") {
            +"add an item"
            clicks handledBy seq.addItem
        }
    }
}
```

In this example `renderEach` uses the equals function to determine, if an item at a given index is still the same. So for the `String` example, render each will not apply any patch to your DOM, when you replace "two" by another instance of String with the same content. It will however remove the `li` representing "two", if you replaced the item it index 1 by "another two" and insert a newly rendered `li` with this text content. So be aware, that the code inside the lambda you give to `renderEach` is executed whenever it is needed to render the DOM representation of a new or changed item.

When dealing with more complex data models this sometimes isn't what you want. When you have a `ToDo` like the following

```kotlin
@Lenses
data class ToDo(
    val id: String = uniqueId(),
    val text: String,
    val completed: Boolean = false
)
```
in your list, which is rendered by 

```kotlin
val toDosStore = storeOf(listOf(ToDo(text = "foo"), ToDo(text = "bar")))

fun main() {
    render {
        section {
            toDosStore.data.renderEach(ToDo::id) { toDo ->
                li {
                    // css is based on https://tailwindcss.com
                    className("line-through".whenever(toDo.completed))
                    +toDo.text
                }
            }
        }
    }
}
```
you just want the `class`-attribute re-rendered when the `ToDo` at a given index is still the same, but just changed the value of its `completed` state. You have to tell `renderEach` how to determine, if an element at an index is still the same although one ot more of its attributes (or sub-entities) have changed. To do so just give it `IdProvider`, a function or lambda that defines how to get the unique id for a certain entity. In this example, we just use the `id`-attribute of the `ToDo`. 

To be able to do two-way-databinding inside `renderEach` you can create a `SubStore` for a given entity conveniently by calling `sub(someEntity)` on a `Store<List<*>>`. This of course only makes sense in combination with an `IdProvider`:

```kotlin
val toDosStore = storeOf(listOf(ToDo(text = "foo"), ToDo(text = "bar")))

fun main() {
    render {
        section {
            ToDosStore.data.renderEach(ToDo::id) { toDo ->
                val entityStore = toDosStore.sub(toDo)
                li {
                    input {
                        value(toDo.text)
                        changes.values() handledBy entityStore.sub(ToDo.text).update
                    }
                }
            }
        }
    }
}
```

If you need two-way-databinding directly on a `Store`'s `data` without any manipulation (filters, maps, etc.) you can call `renderEach(IdProvider)` directly on the `Store`, which will provide the `SubStore` for each element conveniently as the parameter of the render-lambda.

Now you know how to handle all kinds of data and structures in your `Store`s. 
Next, you might want to check whether your model is valid. In fritz2 this is done with [Validation](Validation.html).
