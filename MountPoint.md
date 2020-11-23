---
layout: default
title: MountPoint
parent: Store
nav_order: 5
---
# Mount-Point

A mount-point in fritz2 is an anchor of a `Flow` somewhere in a given structure. It is used as a placeholder when creating the structure. 
Afterwards, each value appearing on the mounted `Flow` will be put into the structure at exactly that position. 

Most of the time you will use mount-point in the browsers DOM, allowing you to mount a `Flow` of `Tag`s to some point in the 
html-structure you are building (e.g. using the `render { }`-context).

@TODO: @STE review
Inside the `RenderContext` opened by `render { }`, a new mount-point is created as a placeholder and 
added to the current parent-element.

@TODO: difference between render{} and renderElement{}
@TODO: describe preserveOrder