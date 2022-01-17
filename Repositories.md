---
layout: default title: Repositories nav_order: 130
---

# Repositories

To connect certain types of backend to your `Store`s, you can make use of fritz2's repositories. fritz2 offers
implementations of the repository-interfaces for two types of backends:

* LocalStorage
* REST
* more like GraphQL are yet to come

The following examples use this `Person` data class:

```kotlin
@Serializable
data class Person(
    val _id: String = "",
    val name: String,
    val age: Int,
    val salary: Int
)
```

## Resource

When defining the data class you want to share using a `Repository`, you must define a `Resource` for it to define, how
to derive a unique id from an instance of your data class and how your data class can be (de-)serialized. You can of
course use [kotlinx-serialization](https://github.com/Kotlin/kotlinx.serialization)
for this. Annotate your data class (here `Person`) with `@Serializable` and then write the following serializer next to
your class (in `commonMain` section):

```kotlin
object PersonResource : Resource<Person, String> {
    override val idProvider: IdProvider<Person, String> = Person::_id

    override fun deserialize(source: String): Person = Json.decodeFromString(Person.serializer(), source)
    override fun serialize(item: Person): String = Json.encodeToString(Person.serializer(), item)

    // needed when using QueryRepository otherwise you can omit
    override fun deserializeList(source: String): List<Person> =
        Json.decodeFromString(ListSerializer(Person.serializer()), source)
    override fun serializeList(items: List<Person>): String =
        Json.encodeToString(ListSerializer(Person.serializer()), items)
}
```

## Repositories

We differentiate two kinds of repositories both using ids of type `I`:

an `EntityRepository` deals with one single entity of a given type `T`. 
It offers the usual CRUD-methods:
* `load(id: I): T`
* `addOrUpdate(entity: T): T`
* `delete(entity: T)`
  
a `QueryRepository` deals with a `List` of instances of a given type `T` and a query object `Q`. 
It offers the following methods to query or manipulate the content of the repository:
* `query(query: Q): List<T>`
* `updateMany(entities: List<T>, entitiesToUpdate: List<T>): List<T>`
* `addOrUpdate(entities: List<T>, entity: T): List<T>`
* `delete(entities: List<T>, id: I): List<T>`
* `delete(entities: List<T>, ids: List<I>): List<T>`
  
  
### EntityRepository

To connect a `Store` with a REST-backend for example, just add the repository-service and use its methods in your
handler:

```kotlin
// use the defined resource from above
val entityStore = object : RootStore<Person>(initialPerson) {

    private val rest = restEntity(PersonResource, http("https://your_api_endpoint"), initialId = "")

    val load = handle<String> { _, id ->
        rest.load(id)
    }

    val addOrUpdate = handle {
        if (it != initialPerson) rest.addOrUpdate(it)
        else it
    }

    val delete = handle {
        rest.delete(it)
        initialPerson
    }

    init {
        syncBy(addOrUpdate)
    }
}
```

The `restEntity` function needs an `initialId` to distinguish add from update operations when calling `addOrUpdate`:
A `POST`request is sent, when the id of the given instance equals `initialID`, otherwise a `PUT` request is used to
update the resource.).

By calling `syncBy` from your init-block the given `Handler` (here `addOrUpdate`) is automatically called on each update
of your `Store`s data, to keep your REST-backend in sync with your local resource.

### QueryRepository

When creating a `QueryRepository`, you can define a type describing the queries which are done by this repository. You
also have to implement a lambda, that defines, how to deal with a concrete instance of this query type:

```kotlin
data class PersonQuery(val namePrefix: String? = null)

val queryStore = object : RootStore<List<Person>>(emptyList()) {
    val localStorage =
        localStorageQuery<Person, String, PersonQuery>(PersonResource, "your prefix") { entities, query ->
            if (query.namePrefix != null) entities.filter { it.name.startsWith(query.namePrefix) }
            else entities
        }

    val query = handle<PersonQuery> { _, query ->
        localStorage.query(query)
    }
    val delete = handle<String> { list, id ->
        localStorage.delete(list, id)
    }

    init {
        query(PersonQuery())
    }
}
```

Of course the receiver and result of this function depend on the concrete implementation you use. For a `RestQuery`, you
have to build and fire the request according to your query-object. If you do not specify a function, all entities of the
defined Resource will be returned.

```kotlin
val restQuery = restQuery<Person, String, PersonQuery>(PersonResource, "https://your_api_endpoint", initialId = "")
```

For more information about the `remote` parameter of `restQuery`, please refer to the docs
section [HTTP-Calls](HttpCalls.html).

# Examples

You can see repositories of all types in action at

* the [repositories example](https://examples.fritz2.dev/repositories/build/distributions/index.html)
* our [Ktor full stack example](https://github.com/jamowei/fritz2-ktor-todomvc)
* the [Spring full stack example](https://github.com/jamowei/fritz2-spring-todomvc) 

