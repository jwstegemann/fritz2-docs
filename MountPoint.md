---
layout: default
title: MountPoint
parent: Store
nav_order: 64
---
# Mount-Point

A mount-point in fritz2 is an anchor of a `Flow` somewhere in a given structure. It is used as a placeholder when creating the structure. 
Afterwards, each value appearing on the mounted `Flow` will be put into the structure at exactly that position. 

Most of the time you will use mount-points in the browser's DOM, allowing you to mount a `Flow` of `Tag`s to some point in the html-structure you are building using for example the `someFlow.render {}` function.

Inside the `RenderContext` opened by `someFlow.render {}`, a new mount-point is created as a placeholder and 
added to the current parent-element. Whenever a new value appears on the `Flow`, the new content is rendered and replaces the old elements.

In this `RenderContext`, any number of root elements can be created (also none).
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
Whenever you are sure that there will be only one root element, you can use the `renderElement {}` function instead: it is optimized for a single element and performs better.
Additionally, you can influence how fritz2 handles the order of elements whenever no `Tag`s exist on the same level as the `render` block, or the order of the elements does not matter: use `preserveOrder = false` to deactivate the ordering of elements to again increase performance. This is highly recommended when re-rendering a lot in your application.

```kotlin
render {
    div {
        div { +"1" }
        flowOf("2").renderElement(preserveOrder = false) {
            div { +it }
        }
        div { +"3" }
    } // renders 1, 3, 2
}
```