---
layout
title
parent
nav_order
---
# Styling DSL and Theming

To make building components with unified styling even faster and simpler, fritz2 offers its own DSL which makes use of constraint-based style props based on scales defined in your theme.

```kotlin
fun RenderContext.headerImage(srcUrl: String, alternativeText: String) {
    ::img.styled {
        boxShadow { flat }
        radius { large }
        width(sm = { small }, md = { smaller })
    } {
        src(srcUrl)
        alt(alternativeText)
    }
}
```

fritz2-styling offers a convenient syntax for adding responsive styles with a mobile-first approach. It uses four breakpoints for the different viewport-sizes.
 
```kotlin
    font-size(sm = { tiny }, lg = { normal })
    width(sm = { full }, lg = { "768px" })
    color { primary }
```

In accordance with mobile-first, the next smaller size is used when no value is given for a particular size. In the example above, `smaller` would be used for `md`, `lg`, and `xl`. If only one value is given, it will be used for all sizes.
 
These centrally defined theme scales for different properties help in achieving a consistent design. Also, simply changing these definitions alters the theme and therefore the look of the application for different requirements. You can define a high contrast theme, or one with large fonts, both of which can be chosen dynamically at runtime.
 
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

# ThemeProvider

* Usage
* Accessing currentTheme


# CSS

* DSL scope
* Link styled-system
* Using CSS
* Pseudos
* Selectors
* Children