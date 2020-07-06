---
layout: default
title: Flows
parent: State Management
nav_order: 6
---
# Stores verbinden

In größeren real-world Anwendungen, die viele verschiedene `Stores` enthalten können, ist es unverzichtbar,
dass diese Stores miteinander kommunizieren können.

Für diesen Fall bietet fritz2 den `EmittingHandler` an, welcher ein spezieller `Handler` ist,
mit dem man nach außenhin Informationen offern kann.

Die Erzeugung dieses `Handler`s erfolgt mit Hilfe der `handleAndOffer` Funktion analog zur `handle` Funktion,
 was das folgende Beispiel zeigt:

```kotlin
val dataStore = object : RootStore<String>("start") {
    val offerLength = handleAndOffer<String, Int> { model, action: String ->
        console.log("new data: $action")
        offer(action.length) // this what I want to offer
        model // don`t change the model
    }
}
```
Der hier dargestellte `EmittingHandler` `offerLength` zählt die Länge des übergebenen Strings `action` und 
offeriert sie nach der Ausgabe auf der Console. Nun kann ein `Handler` aus einem anderen `Store` 
diesen Wert nehmen und verarbeiten. Dazu müssen diese beiden `Handler` aber noch verbunden werden.
Das Verbinden geschieht durch den Aufruf der `handledBy` Funktion, wie das folgende Beispiel zeigt:

```kotlin
val dataStore = ... //see above

val lengthStore = object : RootStore<Int>(0) {
    val handleLength: SimpleHandler<Int> = handle<Int> { model, action: Int ->
       console.log("offered length is: $action")
       action // setting new offered length
    }
}

// don't forget to connect the handlers
dataStore.offerLength handledBy lengthStore.handleLength
```
Nun sind die beiden `Handler` `dataStore.offerLength` und `lengthStore.handleLength` miteinander verbunden, 
sodass sich immer wenn `dataStore.offerLength` aufgerufen wird ein neuer Wert in den `lengthStore` geschrieben wird.
Der Aufruf des `dataStore.offerLength` handlers kann z.B. durch ein `changes.value() handledBy dataStore.offerLength`
in einem `input {}` erfolgen.

Ein Beispiel das dieses Konzept in der Praxis zeigt, ist das [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html).
Dort werden nur `Person`s aus dem einem `Store` dem anderen `Store` geschrieben, wenn diese auch valide sind.
Die Verlinkung der beiden `Handler` erfolgt dabei mit dem Aufruf `personStore.save handledBy listStore.add`.
