---
layout: post
title: 12 Factor App Series - Dependencies
subtitle: Declaration, isolation, and uniformity
comments: true
share-img: /assets/img/12-factor/library.jpg
---

### _Library Dependencies_
One of the most important steps to building a 12 Factor application is ensuring the right tools are used.
Dependencies, or API libraries, allow us to build enterprise grade applications quickly, and it's all on the backs of countless API libraries.
Dependencies unlock the potential of web applications in the modern age, but they need to be declarative, isolated, and
uniform to maintain applications effectively.

![](../assets/img/12-factor/library.jpg)

### _Dependency Declaration_
12 Factor states we need to _declare(s) all dependencies, completely and exactly_ using something called a **manifest**.
In the context of Java programming, a manifest is a way to declare all of our dependent libraries upfront and in one place.
There are several options available in the Java ecosystem, however Maven is what I want to focus on since it is often
the underlying technology of many build tools for Java, including Gradle.

Maven implements a concept called 
[Bill of Materials](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#bill-of-materials-bom-poms),
which gives us a mechanism to declare and manage dependencies across an entire platform.  
If we have a reliance on a subset of dependencies on several applications, 
we can create a **Bill of Material** to manage those versions, and distribute it to each application.
This helps maintain a clean bill of health for every application across the platform.

![](../assets/img/12-factor/library2.jpg)

### _Dependency Isolation_

Dependency managers in Java have a concept called **transitive** dependencies.  

Let us take a look at [transitive relationships](https://en.wikipedia.org/wiki/Transitive_relation) in Set Theory.
This will help us get a better understanding.

**A relation R on a set X is transitive if,**
_for all elements a, b, c in X, whenever R relates a to b and b to c, then R also relates a to c._

Rewritten for Java applications and dependent libraries:

**An dependent library R on an application X is transitive if,**
_for all libraries a, b, c in X, whenever R relates a to b and b to c, then R also relates a to c._

**Simply put, our application is using a 'transitive' dependency if it is using a library not defined in its manifest.**

If we use the transitive property on any of our dependencies, our manifest will be missing the most important thing for upgrading, 
the name of the library! 

Not only will we not know the name of the dependency, but we also won't know the version used, which is simply dangerous.

![](../assets/img/12-factor/train-map.jpg)

**Transitive libraries break 12 Factor design and will lead to long and difficult upgrade paths.**

Maintaining a 'Bill of Materials' that controls and just 'saying no to transitive' dependencies can lead to a happy 
development environment.  
When we put the two concepts together, we end up with an application we can keep up to date with little effort.

Hope you find this informative and see you on the next topic, [12 Factor Config](https://12factor.net/config)!