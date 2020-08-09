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

    val myComponent = render {
        section {
            ul {
                seq.data.each().render { s ->
                    li {
                        button("btn", id = "delete-btn") {
                            text(s)
                            clicks.map { console.log("deleting $s"); s } handledBy seq.deleteItem
                        }
                    }
                }.bind()
            }
            button("button") {
                text("add an item")
                clicks handledBy seq.addItem
            }
        }
    }
```

`each` allows you to use _one-way-databinding_ when working with the elements in your `List`. If you need _two-way-databinding_ (to edit the single elements in a form, for example), just call `each` on the `Store` instead. This gives you a `Seq` of `SubStore`s, one for each element of your `List`. You can use them just like any other `Store` to build a form and bind your data.

There are four flavours of each to chose from to fit your use-case:

* use `Flow<T>.each()` to map each instance of T to your `Tag`s. It uses Kotlin's equality function to determine whether or not two elements are the same, and therefore re-renders the whole content you mapped when an element is changed or moved.

* with `Flow<T>.each(idProvider: (T) -> String)` you can also map each instance of T to your `Tag`s, but it uses the given idProvider to determine whether or not two elements are the same. In your mapping, you can get a `SubStore` for an element using `listStore.sub(id, idProvider)`, so only the parts that actually changed will be re-rendered. Keep in mind: using this flavour without the `SubStore` an element with the same id but different constant will be treated as unchanged and therefore not be rerendered.

* with `Store<List<T>>.each(idProvider: (T) -> String)` you can also map a `SubStore<T>` to `Tag`s, but it uses the given idProvider to determine whether or not two elements are the same`, so only the parts that are bound and  actually changed will be re-rendered. Use this whenever you work on entities that can be identified using some sort of constant id and need two-way-databinding.

* use `Store<List<T>>.each()` to map a `SubStore<T>` to `Tag`s. It uses the list position of the element to determine whether or not two elements are the same. This means that when inserting something into the middle of the list, the changed element AND ALL following elements will be re-rendered. Use this when you work on elements that are not identifiable using a constant id (like simple [String]s) and still need two-way-databinding. 

`store.data.each(...).render {...}` is just a short for `store.data.each(...).map { render {...} }`. 

Like for the single elements in your `List`, you will also need to get `Store`s for elements "hidden" deeper in your [Nested Structures](NestedStructures.html). Let's see how fritz2 can help you here.
