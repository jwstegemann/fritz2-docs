---
layout: default
title: Nested Structures
has_children: true
nav_order: 8
---
# Nested Structures

Most of the time, your model for a view will not be of a simple data-type or a `List`, but a complex entity, like a person having a name, multiple addresses, an email, a date of birth, etc.

In those cases, you will most likely need `Store`s for the single properties of your main entity, and - later on - for the properties of the sub-entity like the street in an address in our example from above.

fritz2 uses a mechanism called `Lens` to describe the relationship between an entity and its sub-entities and properties. So before you continue, please make yourself familiar with [Lenses](Lenses.html) in fritz2.

Having a `Lens` available which points to some specific property makes it very easy to get a `SubStore` for that property from a `Store` of the parent entity:

```kotlin
    val outerStore = RootStore(Outer(Inner("hello"), "world"))
    // remember, the Lenses-object is created by fritz2-optics per package
    val innerStore = outerStore.sub(Lenses.Outer.inner)
```

Now that you can use your `innerStore` exactly like any other `Store` to set up _two-way-databinding_, call `sub(...)` again to access the properties of `Inner`. If a `SubStore` contains a `List`-type, you can of course iterate over it by using `each()` or `eachStore()` like you are used to. It's fully recursive from here on down to the deepest nested parts of your entity.

You can also add `Handler`s to your `SubStore` by simply calling the `handle`-method:

```kotlin
val booleanSubStore = mainStore.sub(someLens)
val switch = booleanSubStore.handle { model: Boolean ->
   !model
}

render {
    ...
        button {
            text("switch state")
            clicks handledBy switch
        }
    ...
}

````

To keep your code well-structured, it is recommended to implement complex logic at your `RootStore` or inherit it by using interfaces. However, the code above is a decent solution for small (convenience-)actions.

Now that you can handle all kinds of data and structures in your `Store`s you might want to check, if they are valid. In fritz2 this is done by [Validation](Validation.html).