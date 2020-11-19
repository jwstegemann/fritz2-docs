---
layout: default
title: Styling
nav_order: 14
---
# Integrated Styling

fritz2 ist im core optimiert für styling durch externes css (siehe [Attributes and CSS](Attributes%20and%20CSS.html). Dies erleichtert die Verwendung von CSS-Frameworks wie [Bootstrap](https://getbootstrap.com/) oder [Tailwind](https://tailwindcss.com/).

Wenn es darum geht dynamische Komponenten effizient zu entwickeln und zu verwenden, stößt dieser Ansatz allerdings an seine Grenzen. Daher bietet fritz2 im Modul `dev.fritz2:styling` die Möglichkeit, css direkt im Kotlin-code zu schreiben und zu verwenden. Dies bietet einige Vorteile:

* Komponenten-Code und Styling zusammengefasst an einer Stelle (vereinfacht Wartung und Fehlersuche)
* automatische Injektion benötigter (und nur der benötigten) CSS-Klassen
* Nutzung von IDE-Features um z.B. herauszufinden, wo eine Klasse genutzt wird, sie umzubenennen, etc.
* dynamische Generierung von CSS-Klassen in Abhängigkeit von Laufzeit-Variablen
* keine Notwendigkeit, CSS-Dateien parallel zum Komponenten-Code zu verteilen

# Verwendung

Um fritz2-styling nutzen zu können, muss lediglich die passende Abhängigkeit im gradle-build ergänzt werden:

```gradle
    val jsMain by getting {
        dependencies {
            implementation("dev.fritz2:styling:<fritz2-version>"))
        }
    }
```

# StyleClass

Das zentrale Element in fritz2-styling ist die `StyleClass`. Eine Instanz entspricht dabei genau einer CSS-Klasse im Stylesheet. Über das `name`-Attribut kann die StyleClass überall dort verwendet werden, wo im core CSS-Klassen als `String benötigt werden: 

```kotlin
    val height = "100%"
    val isFlex = true


    val myStyle: StyleClass = staticStyle("myClassName", """
            min-height: $height;
            flex-direction: row;
            align-items: stretch;
            color: rgb(52, 58, 64);
            display: ${if (ifFlex) "flex" else "block"};
    """)

    div(myStyle.name) {
        +"here I am"
    }
````

Die factory-Methode `staticStyle` compiliert den übergebenen CSS-Code und legt eine Klasse mit dem Namen `myClassName` in einem von fritz2 verwalteten StyleSheet an. fritz2 verwendet [stylis](https://stylis.js.org/) als css-compiler, dessen homepage einen kurzen Überblick über die gegenüber reinem css leicht erweiterte syntax bietet. So ist die Verwendung eingebetter selektoren ebenso möglich wie media-classes und namespaces.

Of course you can use Kotlin variable and even code inside your styles this way. Since this class has a static name your variable cannot change over time. 

Use `style` to create dynamic classes:

```kotlin
fun RenderContext.myComponent(height: Int, isFlex: Boolean) {
    val myStyle: StyleClass = style("myClassName", """
            min-height: ${height}%;
            flex-direction: row;
            align-items: stretch;
            color: rgb(52, 58, 64);
            display: ${if (ifFlex) "flex" else "block"};
    """, "myComponent")

    div(myStyle.name) {
        +"here I am"
    }
}
```

Jedesmal, wenn diese Komponente gerendert wird, erzeugt fritz2 einen neuen Klassennamen, der sich aus dem prefix und einem errechneten hash-wert zusammensetzt, z.B. `myComponent-iHpEb`. Auf diese Weise werden keine zwei Klassen gleichen inhalts erzeugt, sondern bereits bestehende Klassen wiederverwendet, wenn sich der Inhalt nicht geändert hat. Durch verwendung des prefix ist es möglich, semantische bezeichner zu verwenden. So lässt sich im ergebnis leicht erkennen, aus welcher komponente das div erzeugt wurde. 


Durch die Verwendung von [language injections](https://www.jetbrains.com/help/idea/using-language-injections.html) bietet IntelliJ IDEA allen komfort bei der Bearbeitung des css-codes in Kotlin-Dateien.  



