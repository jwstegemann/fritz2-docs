---
layout: default
title: Format
nav_order: 13
---
# Format

In html you can only use `Strings` in your attributes like in the `value` attribute of `input {}`. To use other data 
types in your model you have to specify how to represent a specific value as `String` (e.g. Number, Currency, Date). 
When you work with `input {}` you also need parse the entered text back to your data type. 
fritz2 provides a special function `format` for this, which creates a `Lens<P, String>` for you:

```kotlin
fun <P> format(parse: (String) -> P, format: (P) -> String): Lens<P, String>
```

The following [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) demonstrates its usage: 
```kotlin
import dev.fritz2.lenses.format

object Formats {
    private val dateFormat: DateFormat = DateFormat("yyyy-MM-dd")
    val date: Lens<Date, String> = format(
        parse = { dateFormat.parseDate(it) },
        format = { dateFormat.format(it) }
    )
}
```

When you have created a special `Lens` for your own data type like `Formats.date`, you can then use it to create a new `SubStore`: 
concatenate your Lenses before using them in the `sub()` method e.g. `sub(L.Person.birthday + Fromat.dateLens)` or
call the method `sub()` on the `SubStore` of your custom type `P` with your formatting `Lens` e.g `sub(Format.dateLens)`.

Here is the code from the [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) 
which uses the special `Lens` in the `Formats` object specified above for the `com.soywiz.klock.Date` type:
```kotlin
import com.soywiz.klock.Date
...

val personStore = object : RootStore<Person>(Person(createUUID()))
...
val birthday = personStore.sub(Person.birthday + Format.date)
// or
val birthday = personStore.sub(Person.birthday).sub(Format.date)
...
input("form-control", id = birthday.id) {
    value = birthday.data
    type = const("date")

    changes.values() handledBy birthday.update
}
```
The resulting `SubStore` is a `Store<String>`.

You can of course reuse your custom formatting `Lens` for every `SubStore` of the same type (in this case `com.soywiz.klock.Date`).