---
layout: default
title: MountPoint
parent: Store
nav_order: 5
---
# MountPoint

A `MountPoint`in fritz2 is nothing more than and anchor of a `Flow` somewhere in a given structure. 
The `MountPoint` itself is used like a placeholder when creating the structure. Afterwards each value that appears on the mounted `Flow` will be put into the structure at exactly that position. 

Most of the time you will use DomMountPoints. That means, that you can mount a `Flow` of `Tag`s at some point in a HTML-subtree you are building (using the `render`-context). To do so, you simply have to call the `bind()`-method on some `Flow` of `Tag`s. 

This methods creates a new `MountPoint` as a placeholder and adds it to the actual parent-element. 

Whenever another `Tag` appears on the `Flow` the `DomMountPoint` replaces the last `Tag` by the new one in the child-list of it's parent-element (or simply appends it, if it is the very first value). 
No more magic at work here.

`DomMultiMountPoint` and `AttributeMountPoint` work exactly the same way. We will have a close look later...