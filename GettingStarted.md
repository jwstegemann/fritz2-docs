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

The `index.html`is just a normal web-page. Be sure to include the resulting script-file from your Kotlin/JS-build .
You can mark an element of your choice with an id to later mount your fritz2-components to it:

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
Inside `main`, you might want to create some content by creating a 
[render](https://api.fritz2.dev/core/core/dev.fritz2.dom.html/render.html) context and 
mount it to the DOM of your `index.html`:

```kotlin
fun main() {
    render("#target") {
        h1 { +"My App" }
        div("some-fix-css-class") {
            p(id = "someId") {
                +"Hello World!"
            }
        }
    }
}
```

By calling `render` the way above your content will be mounted to a `HTMLElement` with the `id="target"`. 
If you want to mount your content to the `body` of your `index.html` you can leave this parameter out. 
Instead of using the `selector` string which uses the [querySelector syntax](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector), 
you can also set a `HTMLElement` directly to the `targetElement` parameter. 
Setting the `override` parameter to `false` means that your content will be appended. The default behavior is that all child
elements getting removed before your content is appended to the `targetElement`.

Run the project by calling `./gradlew jsRun` in your project's main directory. Add `--continuous` to enable automatic
building and reloading in the browser after changing your code.

And voilà, you are done! Maybe you would like to create some more versatile [HTML](Attributes%20and%20CSS.html) now?