---
layout: default
title: Lists in a Model
nav_order: 90
---
# Lists as a Model

`Store`s can be created for any type, including lists:

```kotlin
val listStore = storeOf(listOf("a", "b", "c"))
```

It's perfectly valid to render this `data` by iterating over the `List`:

```kotlin
listStore.data.render { list ->
    list.forEach {
        span { + it }
    }
}
```

But keep in mind that this means re-rendering *all* `span`s in this example when the list changes, regardless of how many items you actually changed. This might be what you want for small `List`s, 
for `List`s that rarely change, or for `List`s with a small representation in HTML (like just text per item), etc. However, for `List`s that change more often and/or result in complex HTML-trees per item, this does not perform well.

For those cases, fritz2 offers the method `renderEach {}` which creates a `RenderContext` and mounts its result to the DOM. 
`renderEach {}` works like `render {}`, but it creates a specialized mount-point in order to 
identify elements for re-rendering. This mount-point compares the last version of your list with the 
new one on every change and applies the minimum necessary patches to the DOM.

```kotlin
val seq = object : RootStore<List<String>>(listOf("one", "two", "three")) {
    var count = 0

    val addItem = handle { list ->
        count++
        list + "Yet another item: No. $count"
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
                        clicks.map { console.log("Deleting $s"); s } handledBy seq.deleteItem
                    }
                }
            }
        }
        button("button") {
            +"Add an item"
            clicks handledBy seq.addItem
        }
    }
}
```

In this example, `renderEach` uses the equals function to determine whether an item at a given index is still the same. So for the `String` example, renderEach won't patch your DOM when "two" is replaced by another instance of String with the same content. It will however remove the `li` representing "two" when the item at index 1 is replaced by "another two" and a newly rendered `li` is inserted with this text content. So be aware that the lambda you pass to `renderEach` is executed whenever the DOM representation of a new or changed item is rendered.

When dealing with more complex data models, this sometimes isn't what you want. 

```kotlin
@Lenses
data class ToDo(
    val id: String = uniqueId(),
    val text: String,
    val completed: Boolean = false
)
```
When you have a `ToDo` like this in your list, which is rendered by..

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
.., you just want the `class`-attribute re-rendered when the `ToDo` at a given index is still the same but changed the value of its `completed` state. `renderEach` must be told how to determine whether an element at an index is still the same even though one or more of its attributes (or sub-entities) have changed. To do so, pass an `IdProvider`, a function or lambda that defines how to get the unique id for a certain entity. In this example, we simply use the `ToDo`'s the `id`-attribute. 

For two-way-databinding inside `renderEach`, you can conveniently create a `SubStore` for a given entity by calling `sub(someEntity)` on a `Store<List<*>>`. This of course only makes sense in combination with an `IdProvider`:

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

If you need two-way-databinding directly on a `Store`'s `data` without any manipulation (filters, maps, etc.), call `renderEach(IdProvider)` directly on the `Store`. This will provide the `SubStore` as the render-lambda's parameter for each element.

Now you know how to handle all kinds of data and structures in your `Store`s. 
Next, you might want to check whether your model is valid. In fritz2 this is done with [Validation](Validation.html).
