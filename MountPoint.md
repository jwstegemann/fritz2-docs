---
layout: default
title: MountPoint
parent: Store
nav_order: 5
---
# Mount-Point

A mount-point in fritz2 is an anchor of a `Flow` somewhere in a given structure. It is used as a placeholder when creating the structure. 
Afterwards, each value appearing on the mounted `Flow` will be put into the structure at exactly that position. 

Most of the time you will use DOM-mount-point's, allowing you to mount a `Flow` of `Tag`s to some point in a 
html-structure you are building (using the `render { }`-context).

@TODO: @STE review
Inside the `RenderContext` from the `render { }` method a new mount-point is created as a placeholder and 
adds it to the current parent-element.

@TODO: difference between render{} and renderElement{}