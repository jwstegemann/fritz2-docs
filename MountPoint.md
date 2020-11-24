---
layout: default
title: MountPoint
parent: Store
nav_order: 5
---
# Mount-Point

A mount-point in fritz2 is an anchor of a `Flow` somewhere in a given structure. It is used as a placeholder when creating the structure. 
Afterwards, each value appearing on the mounted `Flow` will be put into the structure at exactly that position. 

Most of the time you will use mount-points in the browser's DOM, allowing you to mount a `Flow` of `Tag`s to some point in the 
html-structure you are building e.g. using the `someFlow.render { }` function.

Inside the `RenderContext` opened by `someFlow.render { }`, a new mount-point is created as a placeholder and 
added to the current parent-element. Whenever a new value appears on the `Flow` the new content is rendered and replaces the old elements.

@Eva
Im diesem `RenderContext` können beliebig viele Root-Element angelegt werden (oder auch keins).
```kotlin
someIntFlow.render {
    if(it % 2 == 0) p { +"is even" }
    // nothing is rendered if odd        
}
// or multiple root elements
someStringFlow.render { name ->
    h5 { +"Your name is:" }
    div { +name }
    hr {}
}
```
Wenn es sicher ist, dass es nur ein Root-Element gibt, kann stattdessen die `renderElement {}` Funktion verwendet werden.
Diese ist für diesen Fall optimiert und bietet eine bessere Performance. Außerdem gibt es die Möglichkeit, wenn sichergestellt
ist, dass neben dem `render`-Block/Scope (auf der selben Ebene) keine anderen `Tag`s existieren oder
die Reihenfolge keine Rolle spielt, die Beibehaltung der Reihenfolge mit dem setzten von `preserveOrder = false` auszuschalten.
Dies ergibt wiederum eine bessere Performance und sollte vor allem da verwendet werden, wo sehr häufig neu gerendert werden muss.
```kotlin
render {
    div {
        div { +"1" }
        flowOf("2").renderElement(preserveOrder = false) {
            div { +it }
        }
        div { +"3" }
    } // render 1, 3, 2
}
```