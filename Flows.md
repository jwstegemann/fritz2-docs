---
layout: default
title: Flows
nav_order: 5
---
#Flows

fritz2 heavily depends on flows, introduced by kotlinx.coroutines.

A Flow is nothing more than a time discrete stream of values.

Like a collection you can use a Flow to represent multiple values, but while using a List for example you can only process all values in it at once. Using a Flow you get the values one by one.

A Flow is build from a source, where the values are created. Your model can be such a source for example, or the events raised by an element. On the other end of the Flow the values are collected one by one by a simple function, that is called once for each element. In between there can be various intermediate actions, like mapping the values (i.e. formatting strings, making HTML-Tags from your model data, etc.), filtering the data, etc.

The clue of those Flows is, that they are cold. That means, that all the intermediate only run, when the are needed. This makes them perfect for fritz2`s use case.

In Kotlin there is another communication model called Channel. They are the hot counterpart to the Flow. fritz2 to uses Channels, too, but they are internal (feed the flows) and just using fritz2 you should stumble upon them.

To get more information about Flow (and if you are interested Channel) and it's API, please have a look at the official documentation.

This is what fritz2 uses to handle the state and events of your app. Now have a closer look at the Store...