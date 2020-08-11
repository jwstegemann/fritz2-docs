---
layout: default
title: Format
nav_order: 13
---
# Format

Sometimes we want to use special data types, for example `com.soywiz.klock.Date` for Dates, 
which supports kotlin-js and other platforms as well. 
However, there is a small problem when using it within an `HTMLInputElement`: it can only handle 
`String` in its `value` attribute, so you have to format your data type to `String` and vice versa. 
fritz2 provides a special function `format` for this, which creates a `Lens<P, String>` for you:

```kotlin
fun <P> format(parse: (String) -> P, format: (P) -> String): Lens<P, String>
```
The following [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) demonstrates its usage: 
```kotlin
import dev.fritz2.lenses.format

object Format {
    private val dateFormat: DateFormat = DateFormat("yyyy-MM-dd")
    val dateLens: Lens<Date, String> = format(
        parse = { dateFormat.parseDate(it) },
        format = { dateFormat.format(it) }
    )
}
```

When you have created a special `Lens` for your own data type like `Format.dateLens`, you can now use them for creating a new `SubStore`: 
call the method `sub()` on the `SubStore` of your custom type `P` with your formatting `Lens` e.g `sub(Format.dateLens)` 
or concatenate your Lenses before using the `Lens` in the `sub()` method e.g. `sub(L.Person.birthday + Fromat.dateLens)`.

Here is the code from the [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) 
which uses the special `Lens` in the `Format` object specified above for the `com.soywiz.klock.Date` type:
```kotlin
import com.soywiz.klock.Date
...

val personStore = object : RootStore<Person>(Person(createUUID()))
...
// creates a FormatStore<Person, Date>
val birthday = personStore.sub(Person.birthday).sub(Format.date)
// or
val birthday = personStore.sub(Person.birthday + Format.date)
...
input("form-control", id = birthday.id) {
    value = birthday.data
    type = const("date")

    changes.values() handledBy birthday.update
}
```
The resulting `SubStore` is basically a `Store<String>`. As you can see in the example above, your internal type 
is converted to a `String` when binding it to the input `value` attribute, using the given `parse: (String) -> P` function
in from the `format()` function. When converting the changed `String` back to a new `Date` object, 
the given `format: (P) -> String` function gets used.

You can of course reuse your custom formatting `Lens` for every `SubStore` of the same type (in this case `com.soywiz.klock.Date`).