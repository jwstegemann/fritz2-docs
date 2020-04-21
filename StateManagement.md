---
layout: default
title: State Management
nav_order: 7
---
#State Management

Building your Store, you can add Handlers to respond to actions and adjust your model accordingly:

val store = RootStore<String>("") {
    val append = handle<String> { model, action -> //action is a String
        "$model$action"
    }

    val clear = handle { model ->
        ""
    }
}
Whenever a String is sent to the append-Handler it updates the model by append the text to it's actual model. clear is a Handle that does not need any information to do his work, so you can just leave out the second parameter.

Since everything in fritz2 is reactive, you won't call the handler directly most of the time but connect a Flow of actions to it:

val stringsToAppend: Flow<String> = ... //we will see, where you get this stream from later on.
store.append <= stringsToAppend
Each Store inherits a Handler called update accepting the same type as the Store as it's action. It just updates the Store's valued to the new value it receives. You can use this handle to conveniently implement two-way-databinding by using the changes event-flow of an input-Tag for example:

val store = RootStore<String>("")

val myComponent = html {
    input {
        value = store.data
        store.update = changes
    }
}
changes in this example is a flow of String created by listening to the Change-Event of the underlying input-element. Whenever this event is raised, a new value appears on the Flow and is processed by the update-Handler of the Store to update the model. Event-flows are available for all HTML5-events.

Of course you can map the elements of the Flow to a specific action-type before you connect it to the Handler. This way you can also add information from the rendering-contect to the action.

You can also use any other source for a Flow like recurring timer events or even external events like web-sockets, local storage, etc.

Next we will have a look at how to use Lists in a model.