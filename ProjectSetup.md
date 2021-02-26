---
layout: default
title: Project Setup
nav_order: 20
---
# Project Setup

To use fritz2, you have to set up a Kotlin multiplatform-project. To do so you can either
* [clone our template from github](https://github.com/jwstegemann/fritz2-template)
* clone the [fritz2-examples](https://github.com/jamowei/fritz2-examples) and copy from one of the sub-projects
* have a look a the [official documentation](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#setting-up-a-multiplatform-project) and include the following plugin:

```gradle
plugins {
    id("dev.fritz2.fritz2-gradle") version "0.9"
}

repositories {
    jcenter()
}

kotlin {
    jvm()
    js().browser()

    sourceSets {
        val commonMain by getting {
            dependencies {
                // implementation("dev.fritz2:components:0.9")
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
```

## Pre-release builds
If you want get the newest snapshot-builds of fritz2 before we release them add the 
following lines to your `build.gradle.kts`:
```gradle
...
repositories {
    jcenter()
    maven("https://oss.jfrog.org/artifactory/jfrog-dependencies") // new repository here
}

kotlin {
    jvm()
    js().browser()

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("dev.fritz2:core:0.10-SNAPSHOT") // add the newer snapshot version here
                // implementation("dev.fritz2:components:0.10-SNAPSHOT")
            }
        }
        ...
    }
}
```
If you find any problems with these snapshot-versions please 
[open an issue](https://github.com/jwstegemann/fritz2/issues/new/choose).