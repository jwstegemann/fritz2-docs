---
layout: default
title: Lists in a Model
nav_order: 8
---
# Lists as a Model

Like for any other type, you can create a `Store` that holds a list:

```kotlin
val listStore = RootStore<List<String>>(listOf("a","b","c"))
```

It is perfectly valid to handle this as seen before: render the `data` by iterating over the `List` (using `forEach` for example). Keep in mind, that this means that your whole `List` is re-rendered whenever an element in your model changes. This might be exactly what you want for small `List`s, `List`s that do not or rarely change, `List`s with a small representation in HTML (just a Test per item), etc.
However, for `List`s that change more often and/or result in complex HTML-trees per item this does not perform well.

For those cases, fritz2 offers the method `renderEach()` which creates a `RenderContext` and mounts its result to the DOM. 
`renderEach` works analogue to `render` and `renderElement`, but it creates a specialized mount-point that compares the last version of your model with the new one on every change to identify, which elements have to be re-rendered and applies the minimal necessary patches to the DOM.

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
}.mount("target")
```

`renderEach{ }` allows you to use _one-way-databinding_ when working with the elements in your `List` out of the box. 
If you need _two-way-databinding_ (to edit the single elements in a form, for example), just call `renderEach` on the `Store<List<T>` instead. 
This gives you a `SubStore` for each element of your `List`. You can use them just like any other `Store` to build a form and bind your data.

in `commonMain`:
```kotlin
@Lenses
data class ToDo(
    val id: String = uniqueId(),
    val text: String,
    val completed: Boolean = false
)
```

in `jsMain`:
```kotlin
val defaultToDos = listOf(ToDo(text = "foo"), ToDo(text = "bar"))
object ToDosStore: RootStore<List<ToDo>>(defaultToDos) {
    // handlers here...
}

render {
    section {
        ToDosStore.renderEach(ToDo::id) { toDo ->
            val textStore = toDo.sub(L.ToDo.text)
            val completedStore = toDo.sub(L.ToDo.completed)
            
            ...
        }
    }
}.mount("target")
```

If you have to manipulate your `Store`'s data-flow before rendering the elements you can do so and call `renderEach` on the resulting `Flow` and get your `Store` by simply calling `myListStore.sub(myElement)` in your `renderEach`.

There are four flavours of `renderEach` to chose from to fit your use-case:

* use `Flow<T>.renderEach { }` to render each instance of T. It uses Kotlin's equality function to determine 
whether two elements are the same, and therefore re-renders the whole content you mapped when an element 
is changed or moved.

* with `Flow<T>.renderEach(idProvider: (T) -> I)` you render each instance of T, too, but it uses the given 
`idProvider` to determine whether two elements are the same. In your mapping, you can get a `SubStore` for an 
element using `listStore.sub(element, idProvider)`, so only the parts of your sub-model that actually changed will be re-rendered. 
Caution: when using this flavour without the `SubStore`, an element with the same id but different content will 
be treated as unchanged and therefore not be re-rendered.

* with `Store<List<T>>.renderEach(idProvider: (T) -> I) { }` you get a `SubStore<T>` to render your items. It uses the given 
`idProvider` to determine whether two elements are the same`, so only the parts that are bound and actually 
changed will be re-rendered. Use this whenever you work on entities that can be identified using some sort of constant 
id and need two-way-databinding.

* use `Store<List<T>>.renderEach()` to render your items using a `SubStore<T>` when you want to use the position of an element in the list to determine 
whether two elements are the same. This means that when inserting something into the middle of the list, the 
changed element and all following elements will be re-rendered. Use this when working on elements that are not 
identifiable using a constant id (like simple [String]s) and still need two-way-databinding. 

Now you can handle all kinds of data and structures in your `Store`s. 
Next you might want to check, if your model is valid. In fritz2 this is done by [Validation](Validation.html).
