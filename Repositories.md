---
layout: default
title: Repositories
nav_order: 12
---
# Repositories

To add some sort of backend to your `Store`s you can make use of fritz2's repositories. fritz2 offers implementations of the repository-interfaces for several types of backends:
* LocalStorage
* REST
* more like WebSockets, GraphQL etc. soon to be added

## Serializer

When you defined you data class wich you want to share by a `Repository`, you must define
a `Serializer` for it first. You can of course use [kotlinx-serialization](https://github.com/Kotlin/kotlinx.serialization)
for it. Therefore, you need to annotate you data class (here `Person`) with `@Serializable` and
then you can write the following:

```kotlin
object PersonSerializer : Serializer<Person, String> {
    override fun read(msg: String): Person = Json.decodeFromString(Person.serializer(), msg)
    override fun readList(msg: String): List<Person> = Json.decodeFromString(ListSerializer(Person.serializer()), msg)
    override fun write(item: Person): String = Json.encodeToString(ToDo.serializer(), item)
    override fun writeList(items: List<Person>): String = Json.encodeToString(ListSerializer(Person.serializer()), items)
}
```

## Resource

Working with repositories requires the definition of a `Resource`:

```kotlin
val personResource = Resource(
    Person::_id, // function to extract the unique id from a given instance
    PersonSerializer, // implementation of Serializer interface to write and read the entity
    Person() // definition of an empty entity (to reset an edit form, e.g.)
)
```

## Repositories

We differentiate two kinds of repositories:

* an `EntityRepository` deals with one single entity of a given type. It offers the usual CRUD-methods:
  * `load(entity, id)`
  * `saveOrUpdate(entity)`
  * `delete(entity)`
  
* a `QueryRepository` deals with a `List` of instances of a given type. It offers the following methods to query or 
manipulate the content of the repository:
  * `query(entities, query)`
  * `addOrUpdate(entities, entity)`
  * `updateMany(entities, entitiesToUpdate)`
  * `delete(entities, id(s)`)
  
  
### EntityRepository

To connect a `Store` with a REST-backend for example, just add the repository-service and use its methods in your handler:

```kotlin
// use the defined resource from above
val entityStore = object : RootStore<Person>(personResource.emptyEntity) {

    private val rest = restEntity(personResource, "https://your_url")

    val load = handle<String> { person, id ->
        rest.load(person, id)
    }

    val saveOrUpdate = handle { rest.saveOrUpdate(it) }

    val delete = handle { rest.delete(it) }
    
    init {
        syncBy(saveOrUpdate)
    }
}
```

`syncBy` automatically calls the given `Handler` on each update of your `Store`.

### QueryRepository

When creating a `QueryRepository` you can define a type describing the queries done by this repository and a function 
how to execute the query defined by a given instance of this query-type. 

```kotlin
data class PersonQuery(val namePrefix: String? = null)

val queryStore = object : RootStore<List<Person>>(emptyList()) {
    val rest = localStorageQuery<Person, String, PersonQuery>(personResource, "your prefix") { entities, query ->
        if (query.namePrefix != null) entities.filter { it.name.startsWith(query.namePrefix) }   
        else entities
    }

    val query = handle(execute = rest::query)
    val delete = handle<String>(execute = rest::delete)

    init {
        action(PersonQuery()) handledBy query
    }
}
```

Of course the receiver and result of this function depend on the concrete implementation you use. 
For a `RestQuery` you have to build and fire the request according to your query-object:

```kotlin
val queryStore = restQuery<Person, String, PersonQuery>(personResource, "your url") { query ->
    get(query.namePrefix?.let {"?name=$it"}.orEmpty())
}
```

If you do not specify a function all entities of the defined Resource will be returned.

# Examples

You can see repositories of all types in action at 
* the [repositories example](https://examples.fritz2.dev/repositories/build/distributions/index.html) 
* our [Ktor full stack example](https://github.com/jamowei/fritz2-ktor-todomvc) 
* the [Spring full stack example](https://github.com/jamowei/fritz2-spring-todomvc) 

