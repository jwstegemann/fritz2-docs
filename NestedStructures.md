---
layout: default
title: Nested Structures
has_children: true
nav_order: 80
---
# Nested Structures

Most of the time, your model for a view will not be of just a simple data-type but a complex entity, like a 
person having a name, multiple addresses, an email, a date of birth, etc.

In those cases, you will most likely need `Store`s for the single properties of your main entity, and - later on - for 
the properties of the sub-entity like the street in an address in our example from above.

fritz2 uses a mechanism called `Lens` to describe the relationship between an entity and its sub-entities and properties. 
So before you continue, please make yourself familiar with [Lenses](Lenses.html) in fritz2.

Having a `Lens` available which points to some specific property makes it very easy to get a `SubStore` for that 
property from a `Store` of the parent entity:

```kotlin
    val personStore = storeOf(Person(Name("first name", "last name"), "more text"))
    // remember, the L-object is created by fritz2-gradle-plugin per package
    val nameStore = personStore.sub(L.Person.name)
```

Now you can use your `nameStore` exactly like any other `Store` to set up _two-way-databinding_, call `sub(...)` 
again to access the properties of `Name`. If a `SubStore` contains a `List`, 
you can of course iterate over it by using `renderEach {}` like described in [Lists as a Model](ListsinaModel.html). 
It's fully recursive from here on down to the deepest nested parts of your model.

You can also add `Handler`s to your `SubStore`s by simply calling the `handle`-method:

```kotlin
val booleanSubStore = parentStore.sub(someLens)
val switch = booleanSubStore.handle { model: Boolean -> 
    !model
}

render {
    button {
        +"switch state"
        clicks handledBy switch
    }
}
````

To keep your code well-structured, it is recommended to implement complex logic at your `RootStore` or inherit it by using interfaces. 
However, the code above is a decent solution for small (convenience-)handlers.

Your real world data model will most certainly contain lists of elements.
Learn how to handle them effectively by reading about [Lists as a Model](ListsinaModel.html)