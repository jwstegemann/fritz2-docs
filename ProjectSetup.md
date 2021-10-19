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
    id("dev.fritz2.fritz2-gradle") version "0.13"
}

repositories {
    mavenCentral()
}

kotlin {
    jvm()
    js(IR) {
        browser()
    }.binaries.executable()

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("dev.fritz2:core:0.13")
                // see https://components.fritz2.dev/
                // implementation("dev.fritz2:components:0.13")
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
If you want to use the newest snapshot-builds of fritz2 before we release them add the 
following lines to your `build.gradle.kts`:

```gradle
plugins {
    id("dev.fritz2.fritz2-gradle") version "0.13"
}

repositories {
    mavenCentral()
    maven("https://s01.oss.sonatype.org/content/repositories/snapshots/") // new repository here
}

kotlin {
    jvm()
    js(IR) {
        browser()
    }.binaries.executable()

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("dev.fritz2:core:0.14-SNAPSHOT") // add the newer snapshot version here
                // implementation("dev.fritz2:components:0.14-SNAPSHOT")
            }
        }
        ...
    }
}
```
If you find any problems with these snapshot-versions please 
[open an issue](https://github.com/jwstegemann/fritz2/issues/new/choose).