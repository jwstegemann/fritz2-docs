---
layout: default
title: MountPoint
parent: Store
nav_order: 5
---
# MountPoint

A `MountPoint` in fritz2 is an anchor of a `Flow` somewhere in a given structure. It is used like a placeholder when creating the structure. Afterwards, each value appearing on the mounted `Flow` will be put into the structure at exactly that position. 

Most of the time you will use `DomMountPoints`, allowing you to mount a `Flow` of `Tag`s to some point in an HTML-subtree you are building (using the `render`-context). To do so, simply call the `bind()`-method on some `Flow` of `Tag`s. 

This method creates a new `MountPoint` as a placeholder and adds it to the current parent-element. 

Whenever another `Tag` appears on the `Flow`, the `DomMountPoint` replaces the last `Tag` with the new one in the child-list of its parent-element (or appends the very first one). No magic here either! 

`DomMultiMountPoint` and `AttributeMountPoint` work exactly the same way. We will have a closer look at those later.