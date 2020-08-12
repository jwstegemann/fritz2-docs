---
layout: default
title: History in Stores
parent: State Management
nav_order: 8
---
# History in Stores

Sometimes you may want to keep the history of values in your `Store`, so you can navigate back in time to build an undo-function or maybe just for debugging...

fritz2 offers a history-service to do so.

```kotlin
val store = object : RootStore<String>("") {
    val history = history<Person>().sync(this)
}
```

This way you bind the history to the updates of your `Store`, so each new value will be added to the history automatically.

Without `sync()`, you can add new entries to the history by yourself by a call to `history.add(entry)` (when performing a specific action for example).

You can access the complete history via it's `Flow` as a `List` of entries. For your convenience `history` also offers
* a `Flow<Boolean>` called `available` representing if entries are available (to show or hide an undo button e.g.)
* a `last()` method to access the latest entry
* a `goBack()` method to get the latest entry and remove it from the history
* a `reset()` method to clear the history

So for a `Store` with a minimal undo function you just have to:

```kotlin
val store = object : RootStore<String>("") {
    val history = history<String>().sync(this)
    
    // your handlers go here (add history.reset() here where suitable)

    val undo = handle {
        history.back()
    }
}

//...

render {
    // insert your form here

    className = store.history.available.map { if (it) "" else "d-none" }
    button("btn btn-danger ml-2") {
        +"undo"
        clicks handledBy EntityStore.undo
    }
}.mount(somewhere)
```

To see a more mature undo function in action, go to our [repositories example](https://examples.fritz2.dev/repositories/build/distributions/index.html).
