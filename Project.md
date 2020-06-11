---
layout: default
title: Project Setup
nav_order: 2
---
# Project Setup

To use fritz2, you have to set up a Kotlin multiplatform-project. To do so you can either

* clone our template-project from [github](https://github.com/jwstegemann/fritz2-template)
* clone the [fritz2-examples](https://github.com/jamowei/fritz2-examples) and copy from one of the sub-projects
* have a look a the [official documentation](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#setting-up-a-multiplatform-project) and include the following plugin:

```gradle
plugins {
    id("dev.fritz2.fritz2-gradle") version "0.5"
}

repositories {
    jcenter()
}

kotlin {
    kotlin {
        jvm()
        js().browser()

        sourceSets {
            val commonMain by getting {
                dependencies {
                    implementation(kotlin("stdlib"))
                }
            }
            val jvmMain by getting {
                dependencies {
                }
            }
            val jsMain by getting {
                dependencies {
                }
            }
        }
    }
}
```

We recommend organizing your source code like this:

* \<project-root\>
  * src
    * commonMain
      * kotlin
        * model.kt (to use the model in client and server)
    * jsMain
      * kotlin
        * \<your base package\>
          * app.kt (or choose any other name)
      * resources
        * index.html

The `index.html`is just a normal web-page. Be sure to include the resulting script-file from your Kotlin/JS-build. Also mark an element of your choice with an id to later mount your fritz2-components to:

```html
<html>
  <body id="target">
    Loading...
    <script src="yourProject.js"></script>
  </body>
</html>
```

`app.kt` is the starting point of your fritz2 app, so make sure it has a `main`-function. Inside `main`, you might want to create some content by creating a `render`-context, adding [some `Tag`s](https://api.fritz2.dev/core/dev.fritz2.dom.html/-html-elements) and mount it to the DOM of your `index.html`:

```kotlin
package <your package>

import dev.fritz2.dom.html.html
import dev.fritz2.dom.mount

fun main() {
    render {
        div("some-fix-css-class") {
            p(id = "someId") {
                text("Hello World!")
            }
        }
    }.mount("target")
}
```

Run the project by calling `./gradlew jsRun` in your project's main directory. Add `--continuous` to enable automatic building and reloading in the browser when you change your code.

And voilà, you are done! Maybe you would like to create some more versatile [HTML](Attributes%20and%20CSS.html) now?  