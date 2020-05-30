---
layout: default
title: Format
nav_order: 12
---
# Format

Sometimes we want to use special data types, for example `com.soywiz.klock.Date` for Dates, 
which supports kotlin-js and other platforms as well. 
However, there is a small problem when using it within an `HTMLInputElement`: it can only handle 
`String` in its `value` attribute, so you have to format your data type to `String` and vice versa. fritz2 provides the special interface 
`Format` for this, which has the following two methods:

```kotlin
interface Format<T> {
    fun parse(value: String): T
    fun format(value: T): String
}
```
The following [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) demonstrates its usage: 
```kotlin
object Format {
    val date = object : Format<Date> {
        // using the standard html date pattern
        private val formatter: DateFormat = DateFormat("yyyy-MM-dd")
        override fun parse(value: String): Date = formatter.parseDate(value)
        override fun format(value: Date): String = formatter.format(value)
    }
}
```
 
When you have implemented `Format` for your own data type, you can create 
a `FormatStore` from your `SubStore`: call the method 
`using(format: Format<T>)` on the `SubStore` of your custom type `T`.

Here is the code from the [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html) 
which uses the `Format` specified above for the `com.soywiz.klock.Date` type:
```kotlin
import com.soywiz.klock.Date
...

val personStore = object : RootStore<Person>(Person(createUUID()))
...
// creates a FormatStore<Person, Date>
val birthday = personStore.sub(Person.birthday) using Format.date
...
input("form-control", id = birthday.id) {
    value = birthday.data
    type = const("date")

    changes.values() handledBy birthday.update
}
```
The resulting `FormatStore` is basically the same as the `SubStore` except that you 
can't call the `sub(lens: Lens<T, X>)` method on it because you already reached the 
end of your data structure tree. As you can see in the example above, your internal type 
is converted to a `String` when binding it to the input `value` attribute, using
 the `format(value: Date)` method from the `Format.date` object. 
 When converting the `String` to a new `Date` object, the `parse(value: String)` method is used.

You can of course reuse your custom `Format` for every `SubStore` of the same type (in this case `com.soywiz.klock.Date`).