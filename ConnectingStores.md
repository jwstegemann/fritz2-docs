---
layout: default
title: Connecting stores
parent: State Management
nav_order: 6
---
# Connecting stores to each other

Most real-world applications contain multiple stores which need to be linked to properly react to model changes.

To make your stores interconnect, fritz2 offers the `OfferingHandler`, a type of handler that doesn't just take data
 as an argument but also offers data on a new `Flow`. The offered data can then be picked up and handled as usual.  

Create such handler by calling the `handleAndOffer()` function instead of the `handle()` function, but add the 
offered data type to the type brackets: 

```kotlin
val personStore = object : RootStore<Person>(Person(...)) {
    val save = handleAndOffer<Person> { person ->
        // saves the current Person
        offer(person) 
        person // return unchanged person
    }
}
```
The `OfferingHandler` named `save` offers the saved `Person` on its `Flow`. 
Another store can be setup to handle this `Person` by connecting the handlers: 
```kotlin
val personStore = ... //see above

val personListStore = object : RootStore<List<Person>>(emptyList<Person>()) {
    val add = handle<Person> { list, person ->
       console.log("add new person: $person")
       list + person
    }
}

// don't forget to connect the handlers
personStore.save handledBy personListStore.add
```
After connecting these two stores via their handlers, a saved `Person` will also be added to the list
 in `personListStore`. All depending components will be updated accordingly.

To see a complete example visit our 
[validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) which uses connected
 stores and validate a `Person` before adding it to a list of `Person`s.
