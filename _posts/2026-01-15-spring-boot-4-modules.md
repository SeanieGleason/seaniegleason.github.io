---
layout: post
title: Modular Spring Boot 4 for Application Framework Teams
subtitle: Spring Boot 4 Architecture and what it means for Application Framework Teams.
share-img: /assets/img/rubix.jpg
---
In 2021, I led the initiative to adopt **Spring Boot** via a custom wrapper project, marking our company's first transition to a fully featured application framework.  By inheriting from Spring Boot, we were able to leverage its robust ecosystem while tailoring the architecture to our specific enterprise needs.  This allowed us to extend **Spring Boot Starters** with internal defaults, security standards, and pre-configured integrations.

We have now officially migrated this architecture to **Spring Boot 4**.

While **Spring Boot 3** moved us to the **Java 17+** baseline and **Jakarta EE**, **Spring Boot 4** takes our developer experience a step further. It optimizes our **cloud-native** footprint through deep **modularization** and **refined GraalVM support**, making our internal wrapper lighter and more transparent than ever.

![](../assets/img/steel-beams.jpg)

## What is changing in Spring Boot 4?
If your unfamiliar with the changes coming to the architecture of Spring Boot 4, give their [press release blog](https://spring.io/blog/2025/10/28/modularizing-spring-boot) a good read.   

The biggest takeaway is the decoupling of the previously monolithic `spring-boot-autoconfigure` module. Each starter now owns its own dedicated **auto-configuration module**.  Previously, any change to the `spring-boot-autoconfigure` module went into every installation of Spring Boot.  Not any longer.

![](../assets/img/rubix.jpg)
## The New `modules/` Directory
Each subdirectory now represents a focused, standalone capability (e.g., modules/persistence, modules/messaging, modules/security).
Each module is responsible for a single concern and exposes only the APIs and auto-configuration pieces needed to integrate with Spring. 

This modular structure has the benefits of **improving maintainability**, enables **more selective dependency usage**, and supports modern requirements such as **faster startup, reduced memory footprint, and native image compatibility**.

## The New `starters/` Directory
The starters have stayed mostly the same, and still act as a convenience layer. It aggregates the most commonly used modules and provides sensible defaults so applications can get up and running quickly with minimal configuration. By depending on the starter, our developers can opt into an **opinionated**, **batteries-included experience** without needing to understand or manage each individual module.

## The New `core/` Directory
n Spring Boot 4, this is where the Bill of Materials (BOM) was moved to centralize dependency and platform management.  If your unfamiliar with its purpose,
the BOM defines the authoritative versions of Spring, third-party libraries, and internal modules used across the entire project. By relocating it into core, dependency alignment is no longer tied to a specific starter or feature set. Instead, it becomes a true platform concern that every module and starter inherits consistently.

## Does it really help?
A resounding yes!

### Stronger modularity
Individual modules can evolve independently while still sharing a common, enforced dependency baseline.

### Clear ownership
Version and dependency management are explicitly separated from feature implementations.

### Reuse beyond starters
Consumers can import the BOM directly when using only specific modules, without pulling in a full starter.

### Future readiness
The structure better supports Gradle version catalogs, Maven platforms, and native/image-focused builds.

## Conclusion
If your already familiar with the architecture patterns of Spring Boot 3, I hope this helped with understanding the changes coming in Spring Boot 4.  
If you made it this far, thanks for reading!
