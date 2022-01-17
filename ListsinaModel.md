---
layout: default
title: Lists in a Model
nav_order: 90
---
# Lists as a Model

Like for any other type, you can create a `Store` that holds a list:

```kotlin
val listStore = storeOf(listOf("a","b","c"))
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
    val id: Int,
    val text: String,
    val completed: Boolean
) {
    companion object
}
```
in your list, which is rendered by 

```kotlin
val toDoListStore = storeOf(listOf(ToDo(1, "foo", false), ToDo(2, "bar", false)))

fun main() {
    render {
        section {
            toDoListStore.data.renderEach(ToDo::id) { toDo ->
                val toDoStore = toDoListStore.sub(toDo, ToDo::id)
                li {
                    val completed = toDoStore.sub(ToDo.completed()).data
                    // css is based on https://tailwindcss.com
                    className(completed.map { if (it) "line-through" else ""})
                    +toDo.text
                }
            }
        }
    }
}
```

you might just want the `class`-attribute to be re-rendered when the `ToDo` at a given index is still the same, but just the value of its `completed` state changed. You have to tell `renderEach` how to determine, if an element at an index is still the same entity although one ot more of its attributes (or sub-entities) have changed by offering an `IdProvider`. An `IdProvider` is a function mapping an entity to unique id of arbitrary type. In this example, we just use the `id`-attribute of the `ToDo`.

Then create a `SubStore` for a given entity conveniently by calling `sub(someEntity, properIdProvider)` on a `Store<List<*>>`. Of course you can do this yourself by mapping the flows if you work on a `Flow<List<*>>` and have no `Store` available or don't want to utilize fritz2's `Lens`es-:

```kotlin
val completed = toDoListStore.data.map { it.find { t -> t.id == toDo.id } ?: false }
```

Regardless if you use a `SubStore` or your mapped `Flow`: be aware that in the example above nothing will happen to your DOM, when the text of a `ToDo` changes, but it is still at the same position in the list. fritz2 determines that still the same entity (identified by the `IdProvider`) is at the same place in the list and therefore the `li` as a whole won't be re-rendered, but just the `class`-attribute (which is mounted to its own mountpoint depending on you `completed`-flow). This might be exactly what you want, but it depends on your use-case.

You can easily do two-way-databinding inside `renderEach` using a `SubStore` created for a particular entity as seen above. This of course only makes sense in combination with an `IdProvider`:

```kotlin
val toDoListStore = storeOf(listOf(ToDo(text = "foo"), ToDo(text = "bar")))

fun main() {
    render {
        section {
            toDoListStore.data.renderEach(ToDo::id) { toDo ->
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

If you need two-way-databinding directly on a `Store`'s `data` without any intermediate operations (filters, maps, etc.) you can call `renderEach(IdProvider)` directly on the `Store`, which will provide the `SubStore` for each element conveniently as the parameter of the render-lambda.

Now you know how to handle all kinds of data and structures in your `Store`s. 
Next, you might want to check whether your model is valid. In fritz2 this is done with [Validation](Validation.html).
