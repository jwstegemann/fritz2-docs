---
layout: default
title: Flows
parent: State Management
nav_order: 6
---
# Connection stores to each other

Most real-world applications contain multiple stores which need to be linked to properly react to model changes.

To make your stores interconnect, fritz2 offers the `EmittingHandler`, a type of handler that doesn't just take data
 as an argument but also emits (offers) data. The offered data can then be picked up and handled by other stores.  

Create an emitting handler by calling the handleAndOffer function just like the regular handle function, but add the 
offered/emitted data type to the type brackets: 

```kotlin
val stringStore = object : RootStore<String>("I want to count the characters in this String.") {
    val offerLength = handleAndOffer<String, Int> { model, input: String -> // <String, Int>: accept String, offer Int
        console.log("new data: $input")
        offer(input.length) // emit length of the input String
        model // return unchanged model
    }
}
```
This `EmittingHandler` named `offerLength` offers the length of the input String after printing the String to the
 console. Another store can be setup to handle this Int after connection the handlers. To do this, call the handledBy
  function: 
 
```kotlin
val stringStore = ... //see above

val lengthStore = object : RootStore<Int>(0) {
    val handleLength = handle<Int> { model, length: Int -> // model is not used and could be replaced by _
       console.log("I picked up this Int: $length")
       length // update length in this Store
    }
}

// don't forget to connect the handlers
stringStore.offerLength handledBy lengthStore.handleLength
```
After connecting these two stores via their handlers, changes to the String in stringStore will also update the Int
 in lengthStore. To call the `dataStore.offerLength` handler, you could use an input element to create a new String
  and handle it with `changes.value() handledBy stringStore.offerLength`, which will then also update the lengthStore.

Our [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) uses linked stores to
 validate a `Person` before adding it to a list of `Person`s.