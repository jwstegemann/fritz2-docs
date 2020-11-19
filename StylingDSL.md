---
layout
title
parent
nav_order
---
# Styling DSL and Theming

Um noch schneller und einfacher Komponenten bauen zu können, die einem einheitlichen styling-sytem folgen bietet fritz2-styling eine eigene DSL making use of constraint-based style props based on scales defined in your theme

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

fritz2-styling offers a convenient syntax for adding responsive styles with a mobile-first approach. Dabei werden vier breakpoints für unterschiedliche viewport-größen verwendet
 
```kotlin
    font-size(sm = { tiny }, lg = { normal })
    width(sm = { full }, lg = { "768px" })
    color { primary }
```
 
 Gemäß dem mobile-first-Ansatz, greift der Wert für die nächst kleinere Größe, ist für eine Größe kein eigener Wert angegeben. Im Beispiel ob gilt `smaller` also für `md`, `lg` und `xl`. Wird nur ein Wert angegeben gilt diese entsprechend für alle Größen. 

Durch die Verwendung von zentral in einem theme definierten scales für unterschiedliche eigenschaften wird ein konsistentes design erreicht. Darüber hinaus kann durch eine einfache änderung der Definitionen im Theme das Aussehen der Anwendung an unterschiedliche Anforderungen angepasst werden. So lässt sich beispielsweise ein theme für hohen Kontrast oder für große Schriftarten definieren. Das Theme kann zur Laufzeit dynamisch gewählt werden. Ein Theme definiert dabei vorbereitete wertebreiche für

* space
* position
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
    
Selbstverständlich können gestylten Elementen weiterhin `baseClass`, `id` und `prefix` übergeben werden

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

