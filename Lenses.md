---
layout: default
title: Lenses
parent: Nested Structures
nav_order: 6
---
#Lenses

A `Lens` is basically a way to describe the relation between an outer entity and and inner entity in a structure.
It focusses the inner entity from the viewpoint of the outer entity - that is where the name is derived from.
Lenses are especially useful when using immutable data-types like fritz2 does.
For this reason a `Lens` needs to know
  * how to get the value of the inner entity from a given instance of the outer entity
  * how to create a new instance of the outer entity (remember: immutable!) as a copy of a given one with just the inner entity changed to a new given value.
In fritz2 a `Lens` is defined by the following interface:
```kotlin
interface Lens<P,T> {
    fun get(parent: P): T
    fun set(parent: P, value: T): P
    fun apply(parent: P, mapper: (T) -> T): P = //is already implemented
}
```
`apply` allows you to change the actual value of the inner entity within one atomic action.

This interface and some convenience-methods are implemented by a separate project called `fritz2-optics` that you inherit a a transitive dependency by using `fritz2-core`.

Of cource you can easily implement this interface by yourself, by just implementing `get()` and `set()`. fritz2 also offers a method called `buildLens()`, that makes it a little less expressive to do so:

```kotlin
val innerLens = buildLens("inner", { it.inner },{ parent, value -> parent.copy(inner = value)})
```

No magic in here. The first parameter allows you to give the `Lens` an id. When you use `Lens`es with `SubStore`s, this id will be used to generate a valid html-id representing the path through your model you can use to identify your elements semantically (for automated ui-tests for example).

If you have deep nested structures or a lot of them it soon gets tiresome to do this manually, though. Therefore fritz2 offers an annotation you can add to your data-class:
```kotlin
@Lenses
data class Outer(val inner: Inner, val value: String)
```
Using a gradle-plugin fritz2 builds a `Lenses`-object per package from these annotations that contains all the `Lenses` you need. They are named exactly like the entities and properties, so it is trivial to use:

```kotlin
val innerLens = Lenses.Outer.inner
```

To use the gradle-plugin from `fritz-optics`, you have to add the following to your gradle-build:
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

When you compile your code containing `@Lenses`, a file named `GeneratedOptics.kt` will be created in your package, that contains the definition of the `Lenses`-object.

Since your other code will depend on these generated files we recommend implementing your model-classes in a separated sub-project. Have a look at our [`nestedmodel`-example](https://github.com/jwstegemann/fritz2/tree/master/examples/nestedmodel), how to do so. This might be a good idea anyway, since you will have to define a multiplatform-project, to be able to share your model and validation code between browser and backend.  