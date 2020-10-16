---
layout: default
title: Websockets
nav_order: 11
---
# Websockets

In fritz2 gibt es die Möglichkeit **Websockets** zu verwenden.
Dazu muss zunächst einmal ein `Socket` erzeugt werden:
```kotlin
val websocket: Socket = websocket("ws://myserver:3333")
```
Bei der Erzeugung können optional noch ein oder mehrere Protokolle 
angegeben werden. Siehe [hier](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/WebSocket)
für mehr Informationen dazu.

Nachdem nun ein Socket erstellt wurde, wird nun mit Hilfe der Methode `connect()`
eine `Session` zum Server aufgebaut. Über diese `Session` können dann Nachrichten 
an den Server gesendet und von diesem empfangen werden. Das Ganze sieht dann so aus:

```kotlin
val session: Session = socket.connect()

// receiving messages from server
session.messages.body.onEach {
  window.alert("Server said: $it")
}.watch() // watch is needed, cause the message is not bind to html

// sending messages to server
session.send("Hello")
```

Die `Session` gibt, wie man oben sehen kann, einen `Flow` von `MessageEvent`s unter `messages`
zurück. Wenn der Server nun Nachrichten an den Client sendet, erscheint ein neues `MessageEvent` auf
diesem `Flow` und mit verschiedenen Methoden kann der Inhalt der Nachricht ausgelesen werden.
Die Art der Nachricht kann eine der Folgenden sein:
* `data: Flow<Any?>`
* `body: Flow<String>`
* `blob: Flow<Blob>`
* `arrayBuffer: Flow<ArrayBuffer>`

Zudem können auch noch weitere für die Darstellung wichtige Informationen aus der `Session` als
`Flow` ausgelesen werden. Dazu zählen:
* `isConnecting: Flow<Boolean>`
* `isOpen: Flow<Boolean>`
* `isClosed: Flow<Boolean>`
* `opens: Flow<Event>`
* `closes: Flow<CloseEvent>`

Wenn die `Session` nicht mehr benötigt wird, kann sie client-seitig geschlossen werden:
```kotlin
session.close(reason = "finished")
```
Dabei kann optional ein Code und Grund für das Schließen der `Session` angegeben werden.
Nach dem Schließen der `Session` entweder durch den Client oder durch den Server können keine Nachrichten
mehr versendet werden. Falls dies doch getan wird, wird eine `SendException` geworfen.

Möchte man den Inhalt eines `Store`s mit dem Server per Websocket synchronisieren, kann man dies mit der Funktion
`syncWith(socket: Socket, resource: Resource<T, I>)` machen. Sie dazu das folgende Beispiel:

```kotlin
data class Person(val name: String, val age: Int, val _id: String = uniqueId())

val personResource = Resource(Person::_id, PersonSerializer, Person("", 0))
val socket = websocket("ws://myserver:3333")

val entityStore = object : RootStore<Person>(personResource.emptyEntity) {
    init {
        syncWith(socket, personResource)
    }

    // your handlers...
}
```

Wenn nun das Model im `Store` geändert wird, wird automatisch das geänderte Model per Websocket an den Server gesendet.
Andersrum kann der Server natürlich auch ein neues Model an den Client senden, welches dann direkt in den `Store` geschrieben wird. 
Dies bietet eine einfache Möglichkeit Daten über einen `Store` automatisch mit dem Server zu synchronisieren.

Want more? Keep on reading about [Routing](Routing.html).