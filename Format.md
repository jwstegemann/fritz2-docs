---
layout: default
title: Format
nav_order: 12
---
# Format

Sometimes you want to use a special data type e.g for Dates the class `com.soywiz.klock.Date`, 
which supports kotlin-js and other platforms as well.There is only one problem 
when you want to use it within a `HTMLInputElement`, because it can only handle 
`String` in its `value` attribute. Therefore, you must format you data type to 
a `String` and vice versa. For doing this fritz2 provide a special interface 
`Format` which has the following two methods:

```kotlin
interface Format<T> {
    fun parse(value: String): T
    fun format(value: T): String
}
```
An example is the `Format` from the [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html):
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
 
When you have implemented your own `Format` for your special data type, you can create 
a `FormatStore` from your normal `SubStore`. For that you must call the method 
`using(format: Format<T>)` on the `SubStore` of your custom type `T`.

Here is the code from the [validation example](https://examples.fritz2.dev/validation/build/distributions/index.html), 
which uses the above specified `Format` for the `com.soywiz.klock.Date` type:
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

    birthday.update <= changes.values()
}
```
The resulting `FormatStore` is basically the same as the `SubStore` except that you 
can't call the `sub(lens: Lens<T, X>)` method on it, because you already reached the 
end of your data structure tree. As you can see in the example above your internal type 
gets converted to a `String` when binding it to the input `value` attribute. For this it 
uses the `format(value: Date)` method from the `Format.date` object. For the other direction 
when converting the `String` to a new `Date` object it uses the `parse(value: String)` method.

Of course, you can reuse your custom `Fromat` for every `SubStore` of the same type, e.g. `com.soywiz.klock.Date`.