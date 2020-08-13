---
layout: default
title: MountPoint
parent: Store
nav_order: 5
---
# MountPoint

A `MountPoint` in fritz2 is an anchor of a `Flow` somewhere in a given structure. It is used as a placeholder when creating the structure. 
Afterwards, each value appearing on the mounted `Flow` will be put into the structure at exactly that position. 

Most of the time you will use `DomMountPoint`s, allowing you to mount a `Flow` of `Tag`s to some point in an 
HTML-subtree you are building (using the `render`-context). To do so, simply call the `bind()`-method on some `Flow` of `Tag`s.

This method creates a new `MountPoint` as a placeholder and adds it to the current parent-element. 

Whenever another `Tag` appears on the `Flow`, the `DomMountPoint` replaces the last `Tag` with the new one in the 
child-list of its parent-element (or appends the very first one). No magic here either!

If you need to mix constant `Tag`s and `DomMountPoint`s within one parent und guarantee the order of children set the 
`preserveOrder`-parameter when binding:
```kotlin
div {
  div { /* ... */ }
  flowOfTags.bind(preserveOrder = true)
  div { /* ... */ }
}
```
Since this temporarily creates a placeholder-element and is not required in most use-cases, it is not 
the default-behaviour for performance-reasons. Of course, you can also wrap your [DomMountPoint]s in a constant [Tag].

`DomMultiMountPoint` and `AttributeMountPoint` work conceptually the same way as `DomMountPoint`s. We will have a closer look at those later.