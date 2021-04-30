---
layout: default
title: Styling DSL
parent: Styling
nav_order: 162
---
# Styling DSL

To make building components with unified styling even faster and simpler, fritz2 offers its own DSL which makes use of 
constraint-based style props based on scales defined in a `Theme`. 
Just use the overloaded factory function to create a HTML tag and specify your concrete 
styling as lambda expression as first parameter:

```kotlin
import dev.fritz2.styling.*

fun RenderContext.headerImage(srcUrl: String, alternativeText: String) {
    img({
        boxShadow { flat }
        radius { large }
        width(sm = { small }, md = { smaller })
    }) {
        src(srcUrl)
        alt(alternativeText)
    }
}
```

**Important**: Add the following import to get the new extension functions: ``import dev.fritz2.styling.*``

Remember that this does not result in inline-styling but rather uses fritz2's `style` function to dynamically create 
css classes in a dynamic style sheet managed by fritz2. See [Styling](Styling.html) fot further details.

fritz2's styling DSL does not aim to support the entirety of the css standard. 
Similar to [Styled System](https://styled-system.com/), it offers fast, type-safe, and themed access to the most important 
properties for adapting components and elements to the special requirements of any situation, and for re-composing 
them time and again.

To remain as flexible as possible, values of properties can alternatively be passed as `String`s, 
like `width { "73%" }`. Additionally, using the `css()` function allows you to set properties that are not part of the DSL:

```kotlin
p({
    css("animation: mymove 5s infinite;")
}) {
    div { + "some animated content"}
}
```

In addition to the usual pseudo elements and classes, the function `children()` gives access to (sub-)selectors:

```kotlin
div({
    children(" > :not(:first-child)") {
        margins { top { large } }
    }
}) {
    div { +"no margin "}
    div { +"margin "}
}
```

You can still pass `baseClass`, `id`, and `prefix` to styled elements, of course.

```kotlin
fun RenderContext.headerImage(srcUrl: String, alternativeText: String) {
    img({
        boxShadow { flat }
        radius { large }
        width(sm = { small }, md = { smaller })
    }, id = "4711", prefix = "headerImage") {
        src(srcUrl)
        alt(alternativeText)
    }
}
```

## Responsiveness

fritz2-styling offers a convenient syntax for adding responsive styles with a mobile-first approach. 
It uses four breakpoints for the different viewport-sizes (sm, md, lg and xl). 
You can set each property independently for these viewport-sizes:
 
```kotlin
{
    fontSize(sm = { tiny }, lg = { normal })
    width(sm = { full }, lg = { "768px" })
    color { primary.mainContrast }
}
```
The concrete definition of the breakpoints (by media-query) is part of each `Theme`.

In accordance with mobile-first, when no value is given for a particular size, the next smaller one will be applied. 
In the example above, the font-size will be `tiny` for `sm` and `md` and `normal` for `lg` and `xl`. 
If only one value is given, it will be used for all sizes.
 
## Theme

The following scales are defined in a theme for different properties to achieve a consistent design. 
Also, simply changing these definitions in the theme alters the whole look of the application and all affected components. 
That way you can easily define themes according to certain requirements like a high contrast theme, or one with large fonts, 
both of which can be chosen dynamically at runtime.
 
A theme defines value ranges for

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

fritz2 provides you with a `DefaultTheme` which is used if you do not specify otherwise (see Current Theme further down below).

## Reusability

You can predefine reusable combinations of properties which are also customizable when specified as functions:

```kotlin
val teaserText: Style<BasicParams> = {
    fontWeight { semiBold }
    textTransform { uppercase }
    textShadow { glowing }
    color { info.mainContrast }
}

render {
    p({
        margins { top { small } }
        teaserText() 
    }) {
        +"myTeaser"
    }
}
```

## Current Theme

Use the `render(someTheme) {}` function at your root to set the `Theme` used to render all subsequent content:

```kotlin
object MyTheme: DefaultTheme() {
    override val name = "MyTheme"

    override val colors = object : Colors by super.colors {
        override val font = "darkblue" // overrides default font color
    }
}

render(MyTheme) {
    p { +"Here is my darkblue text" }
}
```

You can access the current `Theme` at any time by calling `Theme()` inside a `RenderContext`.
Change it at runtime by `Theme.use(myTheme)`. 
See an example for dynamic theme-switching at runtime [here](https://components.fritz2.dev/#Theme).


## Extended Themes

Feel free to extend the theme interface with your own definitions (e.g. corporate design):
```kotlin
interface ExtendedTheme {
    val importantColor: ColorProperty
    val teaserText: Style<BasicParams>
}

object MyTheme : ExtendedTheme, DefaultTheme() {
    override val importantColor = "red"

    override val teaserText: Style<BasicParams> = {
        fontWeight { semiBold }
        textTransform { uppercase }
        letterSpacing { large }
    }
}
```

In render functions, the current theme is accessible via a parameter. 
You can specify the specific sub-type here as well to avoid numerous calls to `Theme()` and casting: 

```kotlin
fun main() {
    render(MyTheme) { theme: MyTheme ->
        p({
            color { theme.importantColor }
            theme.teaserText()
        }) { +"some great looking text" }
    }
}
```