---
layout: default
title: Validation
nav_order: 9
---
# Validation

When accepting user-input, it is a nice idea to validate the data before processing it any further.

To do validation in fritz2, you first have to implement the interface `Validator`. This interface takes three type-parameters:
* the type of data to validate
* a type describing the validation-results (like a message, etc.)
* a type for metadata you want to forward from your `Handler`s to your validation (or `Nothing` if you do not need this)

Now you have to implement the `validation`-method itself:

```kotlin
enum class Severity {
    Info,
    Warning,
    Error
}

data class ValMsg(override val id: String, val severity: Severity, val text: String): Failable {
    override fun isFail(): Boolean = severity > Severity.Warning
}

object EMailValidator: Validator<String, ValMsg, String>() {

    override fun validate(data: String, metadata: String): List<ValMsg> {
        val msgs = mutableListOf<ValMsg>()

        if(data.isEmpty()) {
            msgs.add(ValMsg("empty", Severity.Info, "Please provide some input"))
        }

        if(!data.matches("(^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\\.[a-zA-Z0-9-.]+$)")) {
            msgs.add(ValMsg("not_matched", Severity.Error, "Please correct the email address!"))
        }

        return msgs
    }
}
```
You can structure and implement your concrete validation-rules with everything Kotlin offers.

To use this `Validator`in your `Store`, simply implement the `Validation`-interface by defining a validator for the data-type of your `Store`:

```kotlin
val store = object : RootStore<String>(""), Validation<String, ValMsg, String> {
        override val validator = EMailValidator

        val updateWithValidation = handle<String> { data, newData ->
            if (validate(newData, "update")) newData
            else data
        }
    }
```

When you have a `Store` that implements `Validation`, you can access your validation-results by calling `msgs()`. This gives you a `Flow` of the type you defined as your result-type. You can handle this like any other `Flow` of a `List`, for example by rendering your messages:

```kotlin
render {
    ul {
        store.msgs().each().map {
            render {
                li(it.severity.name.toLowerCase()) {
                    text(it.text)
                }
            }
        }.bind()
    }
}
```


It is recommended to implement your validation-code in a multiplatform-project so you can use it in both browser and backend.
Have a look at a more complete example [here](https://examples.fritz2.dev/validation/build/distributions/index.html).

By the way: fritz2 supports connecting to your (http)-backend from the browser with [Remote Calls](RemoteCalls.html).