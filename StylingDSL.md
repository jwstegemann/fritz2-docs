---
layout: default
title: Styling DSL
parent: Styling
nav_order: 1
---
# Styling DSL and Theming

To make building components with unified styling even faster and simpler, fritz2 offers its own DSL which makes use of constraint-based style props based on scales defined in a `Theme`.

```kotlin
fun RenderContext.headerImage(srcUrl: String, alternativeText: String) {
    (::img.styled {
        boxShadow { flat }
        radius { large }
        width(sm = { small }, md = { smaller })
    }) {
        src(srcUrl)
        alt(alternativeText)
    }
}
```

fritz2-styling offers a convenient syntax for adding responsive styles with a mobile-first approach. It uses four breakpoints for the different viewport-sizes (sm, md, lg and xl):
 
```kotlin
    font-size(sm = { tiny }, lg = { normal })
    width(sm = { full }, lg = { "768px" })
    color { primary }
```
The concrete definition of the breakpoints (by media-query) is part of each `Theme`.

In accordance with mobile-first, when no value is given for a particular size, the next smaller one will be applied. In the example above, `smaller` would be used for `md`, `lg`, and `xl`. If only one value is given, it will be used for all sizes.
 
The following scales are defined in a theme for different properties to achieve a consistent design. Also, simply changing these definitions in the theme alters the whole look of the application and all affected components. That way you can easily define themes according to certain requirements like a high contrast theme, or one with large fonts, both of which can be chosen dynamically at runtime.
 
A theme defines value ranges for..

* spacing
* positions
* fontSizes
* colors
* fonts
* lineHeights
* letterSpacings
* sizes
* borderWidths
* radii
* shadows
* zIndices
* opacities
* gaps

You can still pass `baseClass`, `id`, and `prefix` to styled elements, of course. 

```kotlin
fun RenderContext.headerImage(srcUrl: String, alternativeText: String) {
    ::img.styled({
        boxShadow { flat }
        radius { large }
        width(sm = { small }, md = { smaller })
    }, id = "4711", prefix = "headerImage") {
        src(srcUrl)
        alt(alternativeText)
    }
}
```

## Reusability

You can predefine reusable combinations of properties which are also customizable when specified as functions:

```kotlin
val teaserText: Style<BasicParams> = {
    fontWeight { semiBold }
    textTransform { uppercase }
    textShadow { glowing }
    color { info }
}

render {
    (::p.styled(teaserText)) { +"myTeaser" }
}
```


# Current Theme & Switching

You can access the current `Theme` at any time by calling `Theme()`. Change it by `Theme.use(myTheme)`.

Use the `render(someTheme)` function at your root to allow dynamic theme-switching and automatically re-render your content:

```kotlin
fun main() {
    render(MyTheme()) { theme ->
        div { +"your themed content" }
    }.mount("target")
}

```


## Extensions

Feel free to extend the theme interface with your own definitions:

```kotlin
interface ExtendedTheme {
    val importantColor: ColorProperty
    val teaserText: PredefinedBasicStyle
}

open class MyTheme : ExtendedTheme, DefaultTheme() {
    override val importantColor = "red"

    override val teaserText: PredefinedBasicStyle = {
        fontWeight { semiBold }
        textTransform { uppercase }
        letterSpacing { large }
    }
}
```

In all render functions, the current theme is accessible via parameter. You can specify the specific sub-type here as well: 

```kotlin
fun main() {
    currentTheme = MyTheme()

    renderElement { theme: ExtendedTheme ->
        themeProvider {
            ::p.styled {
                color { theme.importantColor }
                theme.teaserText()
            } { +"some great looking text"}
        }
    }.mount("target")
}
```

# Styling DSL

fritz2's styling DSL does not aim to support the entirety of the CSS standard. Similar to [Styled System](https://styled-system.com/), it offers fast, type-safe, and themed access to the most important properties for adapting components and elements to the special requirements of any situation, and for re-composing them time and again.

To remain as flexible as possible, values of properties can alternatively be passed as `String`s, like `width { "73%" }`. Additionally, using the `css()` function allows you to set properties that are not part of the DSL:

```kotlin
::p.styled {
    css("animation: mymove 5s infinite;")
} {
    div { + "some animated content"}
}
```

In addition to the usual pseudo elements and classes, the function `children()` gives access to (sub-)selectors:

```kotlin
    ::div.styled {
        children(" > :not(:first-child)") {
            margins { top { large } }
        }
    } {
        div { +"no margin "}
        div { +"margin "}
    }
```