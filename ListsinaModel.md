---
layout: default
title: Lists in a Model
nav_order: 7
---
# Lists as a Model

Like for any other type, you can create a `Store` that holds a list:

```kotlin
val listStore = RootStore<List<String>>(listOf("a","b","c"))
```

It is perfectly valid to handle this as seen before: map the `data` to a `Flow` of `Tag`s by iterating over the `List` and bind it to some place in your `render`. However, for `List`s that change more often this does not perform well as the entire list will be re-rendered whenever the model changes.

For those cases, fritz2 offers the method `each()` which creates a `Seq` from your `Flow`. `Seq`s offer intermediate operations that work on your list elements.

Binding a `Seq` in your `render` context works exactly as for a `Flow` by just calling `bind()` on it. But instead of a `DomMountPoint`, a `DomMultiMountPoint` will be created. This kind of `MountPoint` gets a `Flow` of patches as its upstream and is therefore able to change only those DOM-elements that need to be changed (when you add a new element to your List for example or remove one):

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

`renderEach` allows you to use _one-way-databinding_ when working with the elements in your `List`. 
If you need _two-way-databinding_ (to edit the single elements in a form, for example), just call `renderEach` on the `Store<List<T>` instead. 
This gives you a `SubStore`, one for each element of your `List`. You can use them just like any other `Store` to build a form and bind your data.

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

There are four flavours of `renderEach` to chose from to fit your use-case:

* use `Flow<T>.renderEach { }` to map each instance of T to your `Tag`s. It uses Kotlin's equality function to determine 
whether two elements are the same, and therefore re-renders the whole content you mapped when an element 
is changed or moved.

* with `Flow<T>.renderEach(idProvider: (T) -> I) { }` you can also map each instance of T to your `Tag`s, but it uses the given 
`idProvider` to determine whether two elements are the same. In your mapping, you can get a `SubStore` for an 
element using `listStore.sub(element, idProvider)`, so only the parts that actually changed will be re-rendered. 
Caution: when using this flavour without the `SubStore`, an element with the same id but different content will 
be treated as unchanged and therefore not be re-rendered.

* with `Store<List<T>>.renderEach(idProvider: (T) -> I) { }` you can also map a `SubStore<T>` to `Tag`s, but it uses the given 
`idProvider` to determine whether two elements are the same`, so only the parts that are bound and actually 
changed will be re-rendered. Use this whenever you work on entities that can be identified using some sort of constant 
id and need two-way-databinding.

* use `Store<List<T>>.renderEach()` to map a `SubStore<T>` to `Tag`s. It uses the list position of the element to determine 
whether two elements are the same. This means that when inserting something into the middle of the list, the 
changed element and all following elements will be re-rendered. Use this when working on elements that are not 
identifiable using a constant id (like simple [String]s) and still need two-way-databinding. 

Like for the single elements in your `List`, you will also need to get `Store`s for elements "hidden" deeper in your 
[Nested Structures](NestedStructures.html). Let's see how fritz2 can help you here.
