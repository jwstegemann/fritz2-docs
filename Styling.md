---
layout: default
title: Styling
nav_order: 14
---
# Integrated Styling

The fritz2 core is optimized for styling with external css (siehe [Attributes and CSS](Attributes%20and%20CSS.html). This makes it easy to use css frameworks like [Bootstrap](https://getbootstrap.com/) or [Tailwind](https://tailwindcss.com/) to build your fritz2-app.

However, this approach does not yield the best results when trying to efficiently develop dynamic components, which is why fritz2 offers the module `dev.fritz2:styling`. It lets you write and use css directly in your kotlin code, giving you the following advantages: 

* Components code and styling are found in the same place, which makes trouble shooting and maintenance easier
* Automatic injection of the necessary (and only the necessary) css classes
* Allows usage of IDE features (renaming classes, finding usages, ...)
* Dynamically generated css classes depending on runtime variables
* No need to distribute css files with the component code

# Verwendung

To use fritz2-styling, add the following dependency to your gradle build:

```gradle
    val jsMain by getting {
        dependencies {
            implementation("dev.fritz2:styling:<fritz2-version>"))
        }
    }
```

# StyleClass

The key element of fritz2-styling is the `StyleClass`, one instance of which represents exactly one css class in the resulting stylesheet. The `name`-attribute can be used whenever css classes are needed as a `String` in fritz2-core:

```kotlin
    val height = "100%"
    val useFlex = true


    val myStyle: StyleClass = staticStyle("myClassName", """
            min-height: $height;
            flex-direction: row;
            align-items: stretch;
            color: rgb(52, 58, 64);
            display: ${if (useFlex) "flex" else "block"};
    """)

    div(myStyle.name) {
        +"here I am"
    }
````

The factory method `staticStyle` compiles the passed css code and inserts a class named `myClassName` into a stylesheet managed by fritz2. It uses the [stylis] (https://stylis.js.org/) css compiler, which offers a slightly extended syntax over pure css. It allows embedded selectors as well as media classes and namespaces. 

Of course, you can use Kotlin variables and even code inside your styles, but since this class has a static name, your variable cannot change over time. 

Use `style` to create dynamic classes:

```kotlin
fun RenderContext.myComponent(height: Int, isFlex: Boolean) {
    val myStyle: StyleClass = style("""
            min-height: ${height}%;
            flex-direction: row;
            align-items: stretch;
            color: rgb(52, 58, 64);
            display: ${if (isFlex) "flex" else "block"};
    """, "myComponent")

    div(myStyle.name) {
        +"here I am"
    }
}
```
Every time this component is rendered, fritz2 creates a new class name, consisting of the prefix and a calculated hash value (i.e. `myComponent-iHpEb`). This ensures that each created class has unique content, and classes that already exist are reused if the content has not changed. The prefix allows you to use semantic identifiers which makes it easy to see the source component in the resulting HTML. 

IntelliJ uses [language injections](https://www.jetbrains.com/help/idea/using-language-injections.html), making it easy and comfortable to edit css code in kotlin files.



