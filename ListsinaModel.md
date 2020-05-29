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

For those cases, fritz2 offers the method `each()` on a `Flow<List<?>>` which creates a `Seq` from your `Flow`. `Seq`s offer intermediate operations that work on your list elements.

Binding a `Seq` in your `render` context works exactly as for a `Flow` by just calling `bind()` on it. But instead of a `DomMountPoint`, a `DomMultiMountPoint` will be created. This kind of `MountPoint` gets a `Flow` of patches as its upstream from the `Seq` and is therefore able to change only the DOM-elements that need to be changed (when you add a new element to your List for example or remove one):

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
                seq.data.each().map { s ->
                    render {
                        li {
                            button("btn", id = "delete-btn") {
                                text(s)
                                clicks.map { console.log("deleting $s"); s } handledBy seq.deleteItem
                            }
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

So, `each()` allows you to use _one-way-databinding_ when working with the elements in your `List`. If you need _two-way-databinding_ (to edit the single elements in a form, for example), just call `eachStore()` on the `Store` instead. This gives you a `Seq` of `SubStore`s, one for each element of your `List`. You can use them just like any other `Store` to build a form and bind your data.
If you have to further process your data-`Flow` before, you can still use `each()` and call `sub()` inside the each body with the element to get the `Store` representing that element in your list.

Like for the single elements in your `List`, you will also need to get `Store`s for elements "hidden" deeper in your [Nested Structures](NestedStructures.html). Let's see how fritz2 can help you here.
