---
layout: default
title: Getting Started
nav_order: 30
---
# Getting Started

We recommend organizing your source code like this:

* \<project-root\>
  * src
    * commonMain
      * kotlin
        * \<your base package\>
          * model.kt (to use the model in client and server)
    * jsMain
      * kotlin
        * \<your base package\>
          * app.kt (or choose any other name)
      * resources
        * index.html

The `index.html`is just a normal web-page. Be sure to include the resulting script-file from your Kotlin/JS-build 
and mark an element of your choice with an id to later mount your fritz2-components to:

```html
<html>
  <head>
    <title>Your title</title>
  </head>
  <body id="target">
    Loading...
    <script src="yourProject.js"></script>
  </body>
</html>
```

`app.kt` is the starting point of your fritz2 app, so make sure it has a `main`-function. 
Inside `main`, you might want to create some content by creating a `render`-context, 
adding [some `Tag`s](https://api.fritz2.dev/core/core/dev.fritz2.dom.html/-render-context/index.html) and 
mount it to the DOM of your `index.html`:

```kotlin
fun main() {
    render {
        h1 { +"My App" }
        div("some-fix-css-class") {
            p(id = "someId") {
                +"Hello World!"
            }
        }
    }
}
```

Run the project by calling `./gradlew jsRun` in your project's main directory. Add `--continuous` to enable automatic 
building and reloading in the browser after changing your code.

And voilà, you are done! Maybe you would like to create some more versatile [HTML](Attributes%20and%20CSS.html) now?