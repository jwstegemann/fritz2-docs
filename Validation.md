---
layout: default
title: Validation
nav_order: 100
---
# Validation

When accepting user-input, it is a nice idea to validate the data before processing it any further.

Hint: If you plan to use our [components](ComponentLibrary.html), you might want to check out the
[guide](https://components.fritz2.dev/#FormControl) specially tailored to the component's validation features.

To do validation in fritz2, you first have to implement the interface `Validator`. 
This interface takes three type-parameters:
* the type of data to validate
* a type describing the validation-results (like a message, etc.)
* a type for metadata you want to forward from your `Handler`s to your validation (or `Unit` if you do not need this)

Put your validation code in the `commonMain` section of your multiplatform-project where your data models
are located as well. Code in `commonMain` can be used in `jsMain` (frontend) and `jvmMain` (backend). 

In the `commonMain` section write something like this:
```kotlin
@Lenses
data class Person(
    val name: String,
    val age: Int
)

enum class Severity {
    Info,
    Warning,
    Error
}

data class Message(val id: String, val severity: Severity, val text: String): ValidationMessage {
    override fun isError(): Boolean = severity > Severity.Warning
}

class PersonValidator : Validator<Person, Message, String>() {

    override fun validate(data: Person, metadata: String): List<Message> {
        val msgs = mutableListOf<Message>()
        val inspector = inspect(data)
        
        val name = inspector.sub(L.Person.name)
        if(name.data.trim().isBlank()) {
            msgs.add(Message(name.id, Severity.Error, "Please provide a name"))
        }

        val age = inspector.sub(L.Person.age)
        if(age.data < 1) {
            msgs.add(Message(age.id, Severity.Error, "Please correct the age"))
        } else if(age.data > 100) {
            msgs.add(Message(age.id, Severity.Warning, "Is the person really older then 100 years‽"))
        }

        return msgs
    }
}
```
You can structure and implement your concrete validation-rules with everything Kotlin offers. 
Also, for getting the same ids as with using `sub()` method on the `SubStore`s, we create a new `Inspector` with the
`ìnspect()` function. The `Inspector` can then navigate through the model by calling the `sub()` method with the corresponding `Lens`.
The resulting `SubInspector`s then have two attributes `data` and `id`. The first one gives you the current data and 
the second one the corresponding id to it.

Now you can use the `PersonValidator` in your `jsMain` section:

```kotlin
val store = object : RootStore<Person>(Person("Chris", 42)) {
    val validator = PersonValidator()

    val updateWithValidation = handle<Person> { oldPerson, newPerson ->
        if (validator.isValid(newPerson, "update")) newPerson else oldPerson
    }
}
```

You can access your validation-results with `Validator.data`. 
This gives you a `Flow<List<M>` where `M` is your `ValidationMessage`-type. 
You can also use the `current` property to get the current `List` of your `ValidationMessage`.
You can handle the `Flow` of your messages like any other `Flow` of a `List`, 
for example by rendering your messages to html:

```kotlin
// create some messages
store.updateWithValidation(Person("", 101))

render {
    ul {
        store.validator.data.renderEach(Message::id) {
            li(baseClass = it.severity.name.toLowerCase()) {
                +it.text
            }
        }
    }
}
```

Have a look at a more complete example [here](https://examples.fritz2.dev/validation/build/distributions/index.html).

By the way: fritz2 supports connecting to your (http)-backend from the browser with [Http Calls](HttpCalls.html).
