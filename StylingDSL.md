---
layout: default
title: Styling DSL
parent: Styling
nav_order: 1
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

fritz2-styling offers a convenient syntax for adding responsive styles with a mobile-first approach. It uses four breakpoints for the different viewport-sizes (sm, md, lg and xl):
 
```kotlin
    font-size(sm = { tiny }, lg = { normal })
    width(sm = { full }, lg = { "768px" })
    color { primary }
```

In accordance with mobile-first, when no value is given for a particular size, the next smaller one will be applied. In the example above, `smaller` would be used for `md`, `lg`, and `xl`. If only one value is given, it will be used for all sizes.
 
The following scales a defined in a Theme for different properties to achieve a consistent design. Also, simply changing these definitions in the theme alters the whole look of the application and all used compoentents. That way you can easily define themes according to certain requirements like a high contrast theme, or one with large fonts, both of which can be chosen dynamically at runtime.
 
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

To use themed components in your application, just add a `themeProvider` component in your main render-function wrapping your content:

```kotlin
fun main() {
    renderElement {
        themeProvider {
            div { +"your themed content" }
        }
    }.mount("target")
}

```

## access current Theme

Auf das aktuelle Theme kann man über die Funktion `Theme()` zugreifen. Mittels `Theme.use(myOtherTheme)` can das aktuelle Theme verändert werden.

## Wiederverwendung

Kombinationen von Properties können vordefiniert und wiederverwendet werden, als Funktion auch parametrisierbar:

```kotlin
val teaserText: Style<BasicParams> = {
    fontWeight { semiBold }
    textTransform { uppercase }
    textShadow { glowing }
    color { info }
}

render {
    ::p.styled(teaserText) { +"myTeaser" }
}
```



## Erweiterungen

Natürlich kann man das Theme-Interface auch um eigene Definitionen erweitern:

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

In den render-Funktionen lässt sich auf das aktuelle Theme per Parameter zugreifen. Hier kann auch gleich der richtige Sub-Typ angegeben werden:

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

Die Styling-DSL von fritz2 hat nicht zum Ziel den kompletten Umfang von CSS abzubilden. Angelehnt an [Styled System](https://styled-system.com/) bietet sie schnellen, typsicheren und themed access (kann ich auf deutsch nicht sagen ;-)) auf die wichtigsten Properties, um die vorhandenen Komponenten und Elementen an die speziellen Anforderungen einer bestimmten Situation anzupassen und sie immer neu zu komponieren (bitte mit compose übersetzen. ist eine Anspielung für JetBrains).

Um dennoch möglichst flexibel zu bleiben, ist es an allen Stellen möglich beliebige Werte für den value einer property als `String` zu anzugeben, wie `width { "73%" }`. Darüber hinaus kann mit der funktion `css()` eine beliebige property beeinflusst werden

Neben den üblichen Pseudo-Elementen und Klassen steht die Funktion `children()` zur Verfügung, um beliebige (Sub-) Selektoren nutzen zu können:

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