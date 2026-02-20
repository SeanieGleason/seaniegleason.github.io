---
layout: post
title: Creating your own Gradle Plugin Configurations
subtitle: Designing plain Objects In the Gradle Lifecycle using Domain Object Containers
share-img: /assets/img/kotlin.jpg
---

![](../assets/img/kotlin.jpg)

## Gradle's Dependency Injection Explained

If your unfamiliar with Dependency Injection, its the "philosophy as code" where anything that asks to be created, is
required to, upfront, ask for everything it needs to live.

This forces a diligent creation strategy where you no longer allow multiple versions of the same thing be created over
and over; you seek to reuse a single item for everyone who needs it.

In Gradle's API, the `NamedDomainObjectContainers` builder and the `Extensions` container are mechanisms to implement
Dependency Injection in the Gradle Lifecycle.

Both are complimentary to one another and help ensure your classes are singletons and you don't repeat yourself.

## The base Interface - `NamedDomainObjectContainers`

The first part of the API is the class builder `NamedDomainObjectContainers`. for creating any custom service classes.
It's only requirement is you implement a name.

```kotlin
abstract class Greetings @Inject constructor(private val name: String) {

    abstract val hello: Set<String> = setOf(
        "Hello",
        "Hola",
        "नमस्ते",
        "Bonjour",
        "Hallo",
        "Ciao",
        "Olá",
        "Привет",
        "你好",
        "こんにちは",
        "안녕하세요"
    )

    fun getName(): String = name
} 
```

#### It gives our class a `square` shape to fit into a `square` hole

## The Extension API

The `project.extensions` api is our object junk drawer and is able to maintain configurability through the
`build.gradle.kts`, i.e.:

```kotlin
greetings {
    hello = "Hello"
})
```

While also ensuring we only ever create a singular instance. e.g.,

```kotlin
class GreetingPlugin @Inject constructor(private val objectFactory: ObjectFactory) : Plugin<Project> {
    override fun apply(project: Project) {
        val greetings = objects.domainObjectContainer(Greetings::class.java)
        val greetings2 = objects.domainObjectContainer(Greetings::class.java)

        project.extensions.extraProperties.set("greetings", greetings)
        project.extensions.extraProperties.set("greetings", greetings2) //fails...
    }
}
```

## The ObjectFactory

What if you don't need to expose an API but still want to use Gradle's Dependency Injection framework to ensure
singleton's?

That's where the ObjectFactory and the `project.extensions.extraProperties` container come in.

The `ObjectFactory` is the main engine behind the Dependency Injection mechanisms in gradle. It is great for creating
general purpose services and injecting Lazy Properties (Property<T>), Domain Containers, and just general configuration
code into a plugins' lifecycle.

It allows Gradle to "decorate" other plugins and classes while not requiring an exposable API through your
`build.gradle.kts`. e.g.,

```kotlin
class GreetingPlugin @Inject constructor(private val objectFactory: ObjectFactory) : Plugin<Project> {
    override fun apply(project: Project) {
        val greetings = objects.domainObjectContainer(Greetings::class.java)

        project.extensions.extraProperties.set("greetings", greetings)
    }
}
```

## Conclusion

That's it!  

Hopefully this helps you write your next framework plugin or add onto your existing plugins your writing.
If you made it this far, thanks and hope to see you in the next article! 
