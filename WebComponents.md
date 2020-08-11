---
layout: default
title: Web Components
nav_order: 14
---
# Use and build WebComponents

With fritz2, you can easily use [WebComponents](https://webcomponents.org) in any html-context:

```kotlin
render {
    div("weather-card") {
        h2 { city.bind() }
        custom("m3-stars") {
            attr("max", "5")
            attr("current", "3.5")
        }
        // ...
    } 
}
```

Before you can use a custom element, you have to add the component to your site's scripts.
One way is adding a script link pointing to the component which is hosted somewhere you can access it:
```html
<script type="module" src="https://unpkg.com/@mat3e-ux/stars"></script>
```

If the component you want to use is published on [npm](https://www.npmjs.com/), you can add it as a dependency in your Gradle-build:

```gradle
dependencies {
    implementation(kotlin("stdlib-js"))
    // ...
    implementation(npm("@mat3e-ux/stars"))
}
```

... and import it in your Kotlin-Code:

```kotlin
@JsModule("@mat3e-ux/stars")
@JsNonModule
abstract external class Stars() : HTMLElement
```

Depending on how the component is built, you might have to register it with the browser:

```kotlin
fun main(args: Array<String>) {
    window.customElements.define("m3-stars", Stars::class.js.unsafeCast<() -> dynamic>())
    // ...
}
```

We cannot provide typesafe attributes for custom elements, but you can implement a `Tag` and provide an extension function for `HtmlElements`:

```kotlin
class M3Stars() : Tag<HTMLElement>("m3-stars"),
    WithText<HTMLButtonElement> {
    var max: Flow<Int> by AttributeDelegate
    var current: Flow<Float> by AttributeDelegate
}

fun m3Stars(content: M3Stars.() -> Unit): M3Stars
    register(M3Stars(), content)
```

## Build a WebComponent

To build a WebComponent with fritz2, two steps are neccessary. First, implement your WebComponent-class: 

```kotlin
class MyComponent : WebComponent<HTMLParagraphElement>() {
    override fun init(element: HTMLElement, shadowRoot: ShadowRoot): Tag<HTMLParagraphElement> {
        linkStylesheet(shadowRoot, "./myStyles.css")
        // setStylesheet(shadowRoot, """p { border: 1px solid red; }""")
   
        // create your Stores, etc.
        return render {
            p() {
                text("I am a WebComponent")
            }
        }
    }
}
```

Next, register your component:

```kotlin
fun main(args: Array<String>) {
    registerWebComponent("my-component", MyComponent::class)
}
```

To observe one or more arguments, just add them to the registration:

```kotlin
    registerWebComponent("my-component", MyComponent::class, "first-attr", "second-attr")
```

You can then use the values of these observed attributes in your init-method as a `Flow`:

```kotlin
val first = attributeChanges("first-attr")

render {
    //...
    first.bind()
}
```

To react to the lifecyle of your component, you can override the according methods from the specification.

Packaging (i.e. as an npm-package) and publishing is out of scope of this documentation.

To see it in action, please have a look at [this](https://examples.fritz2.dev/webcomponent/build/distributions/index.html) more complex example.

