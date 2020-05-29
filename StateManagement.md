---
layout: default
title: State Management
nav_order: 6
---
# State Management

Building your `Store`, you can add `Handler`s to respond to actions and adjust your model accordingly:

```kotlin
val store = object : RootStore<String>("") {
    val append = handle<String> { model, action -> //action is a String
        "$model$action"
    }

    val clear = handle { model ->
        ""
    }
}
```
Whenever a `String` is sent to the `append`-`Handler`, it updates the model by appending the text to its current model. `clear` is a `Handler` that doesn't need any information to do its work, so the second parameter can be omitted.

Since everything in fritz2 is reactive, most of the time you want to connect a `Flow` of actions to the `Handler` instead of calling it directly:

```kotlin
val stringsToAppend: Flow<String> = ... // the source of this stream is set later
stringsToAppend handledBy store.append
```

Each `Store` inherits a `Handler` called `update`, accepting the same type as the `Store` as its action. It updates the `Store`'s value to the new value it receives. You can use this handler to conveniently implement _two-way-databinding_ by using the `changes` event-flow of an `input`-`Tag`, for example:

```kotlin
val store = RootStore<String>("")

val myComponent = render {
    input {
        value = store.data
        changes.values() handledBy store.update
    }
}
```

`changes` in this example is a flow of events created by listening to the `Change`-Event of the underlying input-element. Calling `values()` on it extracts the current value from the input.
Whenever such an event is raised, a new value appears on the `Flow` and is processed by the `update`-Handler of the `Store` to update the model. Event-flows are available for [all HTML5-events](https://api.fritz2.dev/fritz2/io.fritz2.dom/-with-events/).
There are some more [convenience functions](https://api.fritz2.dev/fritz2/io.fritz2.dom/) to help you to extract data from an event or control event-processing.

You can map the elements of the `Flow` to a specific action-type before connecting it to the `Handler`. This way you can also add information from the rendering-context to the action. 
You may also use any other source for a `Flow` like recurring timer events or even external events like web-sockets, local storage, etc.

Next we will have a look at how to use [Lists as a model](ListsinaModel.html).