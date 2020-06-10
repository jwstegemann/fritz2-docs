---
layout: default
title: Lenses
parent: Nested Structures
nav_order: 8
---
# Lenses

A `Lens` is basically a way to describe the relation between an outer and inner entity in a structure.
It focuses on the inner entity from the viewpoint of the outer entity, which is how it got its name.
Lenses are especially useful when using immutable data-types like fritz2 does.
A `Lens` needs to handle the following: 

  * Getting the value of the inner entity from a given instance of the outer entity
  * Creating a new instance of the outer entity (immutable!) as a copy of a given one with a different value only for the inner entity

In fritz2, a `Lens` is defined by the following interface:
```kotlin
interface Lens<P,T> {
    fun get(parent: P): T
    fun set(parent: P, value: T): P
    fun apply(parent: P, mapper: (T) -> T): P = //is already implemented
}
```
`apply` allows you to change the current value of the inner entity within one atomic action.

You can easily use this interface by just implementing `get()` and `set()`. fritz2 also offers the method `buildLens()` for a short-and-sweet-experience:

```kotlin
val innerLens = buildLens("inner", { it.inner }, { parent, value -> parent.copy(inner = value) })
```

No magic there. The first parameter sets an id for the `Lens`. When using `Lens`es with `SubStore`s, the id will be used to generate a valid html-id representing the path through your model. This can be used to identify your elements semantically (for automated ui-tests for example).

If you have deep nested structures or a lot of them, you may want to automate this behavior. fritz2 offers an annotation you can add to your data-class:
```kotlin
@Lenses
data class Outer(val inner: Inner, val value: String)
```
Using an annotation-processor, fritz2 builds a `Lenses`-object per package from these annotations which contains all the `Lenses` you need. They are named exactly like the entities and properties, so it's easy to use:

```kotlin
val innerLens = Lenses.Outer.inner
```

You can see it in action at our [`nestedmodel`-example](https://examples.fritz2.dev/nestedmodel/build/distributions/index.html).

To make this work your annotated classes have to be in your `commonMain`-SourceSet. If you need to use the generated objects in your `commonMain` (to implement a `Validator` you can use on client and server i.e.), you have to set up a subproject for your model. Have a look at the [`validation`-example](https://examples.fritz2.dev/validation/build/distributions/index.html) to see how to set it up.

This will also help you define a multiplatform-project to be able to share your model and validation code between browser and backend.  