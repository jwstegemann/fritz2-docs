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

The interface and some convenience-methods are implemented by a separate project called `fritz2-optics` that are inherited as a transitive dependency by using `fritz2-core`.

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
Using a Gradle-plugin, fritz2 builds a `Lenses`-object per package from these annotations which contains all the `Lenses` you need. They are named exactly like the entities and properties, so it's easy to use:

```kotlin
val innerLens = Lenses.Outer.inner
```

To use the Gradle-plugin from `fritz-optics`, you have to add the following lines to your Gradle-build:
```gradle
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath("io.fritz2.optics:plugin:0.1")
    }
}

apply(plugin = "io.fritz2.optics")
```

When you compile your code containing `@Lenses`, a file named `GeneratedOptics.kt` will be created in your package which contains the definition of the `Lenses`-object.

Since your other code will depend on these generated files, we recommend implementing your model-classes in a separated sub-project. Have a look at our [`nestedmodel`-example](https://api.fritz2.dev/fritz2/io.fritz2.dom.html/-html-elements) to find out how to do so. This will also help you define a multiplatform-project to be able to share your model and validation code between browser and backend.  