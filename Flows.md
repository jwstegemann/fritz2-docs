---
layout: default
title: Flows
parent: Store
nav_order: 5
---
# Flows

fritz2 heavily depends on flows, introduced by [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines).

A `Flow` is nothing more than a time discrete stream of values.

Like a collection you can use a `Flow` to represent multiple values, but unlike using a `List` for example you get the values one by one. In fritz2 we heavily use `Flow`s to represent values, that change over time (your data-model for example).

A `Flow` is built from a source, where the values are created. Your model can be such a source for example, or the events raised by an element. On the other end of the `Flow` the values are collected one by one by a simple function, that is called once for each element. In between there can be various intermediate actions, like mapping the values (i.e. formatting strings, making HTML-Tags from your model data, etc.), filtering the data, etc.

The clue of those `Flow`s is, that they are _cold_. That means, that all the calculations only run, when the result is needed. This makes them perfect for fritz2`s use case.

In Kotlin there is another communication model called `Channel`. It is the _hot_ counterpart of the `Flow`. fritz2 to uses `Channel`s, too, but they are internal (feed the flows) and just by using fritz2 you should not stumble upon them. 

To get more information about `Flow` (and if you are interested `Channel`) and it's API, please have a look at the [official documentation](https://kotlinlang.org/docs/reference/coroutines/flow.html).

This is what fritz2 uses to handle the state and events of your app. Now have a closer look at the [Store](Store.html)...