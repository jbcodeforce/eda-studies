# Introduction

!!! Info "Site updated 07/25/2023"
    - Add Choreography and orchestration introduction in design pattern
    - Add Scatter-gather in design pattern
    - Add Kafka connect content
    - Add event sourcing 
    - Tune Saga pattern content

## Why this site?

IT architects have a lot of initiatives around cloud adoption, and refactoring existing applications to distributed microservices as the values are no more to proof those days, but more the how to do it. So this web site is like a global book on event-driven architecture adoption. Instead of writing a book, that is static by design, I prefer to do this web site so I can adapt content and add new chapters while I learn new things in this exiting domain.

## Target audiance

Overall, event-driven architecture offers numerous benefits, including improved scalability, performance, loose coupling, real-time capabilities, support for microservices, event sourcing, and flexible integration. These advantages make it an attractive option for audiences interested in building modern, responsive, and scalable applications.

I see Entreprise Architects, Solution Architects, and developer leaders will be interested in this content.

## What kind of problems are we trying to solve?

Event-driven architecture is a very loaded term, those days, and very oriented by the software vendor product capabilities. This site will have some implementation done with certain products, but it tries to be product agnostics and present what a EDA should be.

When moving to cloud native implementation architects need to address:

* scalability and Performance,
* the Microservice and distributed system, adoption and well designed with good granularity,
* coupling of services with point to point synchronous communication brings some implementation challenges and frictions for changes,
* the support for real-time processing to act on the data as early as data created, 
* considering events as a source of truth to get eventual consistent systems, be more resilient, 
* how to get started and applied proved methodology.

## At the highest level EDA helps

For Scalability and Performance, Event-driven architecture enables applications to scale easily and handle high loads efficiently. It allows for asynchronous processing, where components react to events as they occur, leading to improved performance and responsiveness.

For loose coupling: Event-driven architecture promotes loose coupling between different components of a system. Each component communicates through events, which reduces dependencies and enables easier maintenance, updates, and scalability. This flexibility appeals to developers and architects who value modularity and extensibility.

Supporting Real-time and Reactive Systems: Event-driven architecture is well-suited for building real-time and reactive systems. By reacting to events as they happen, applications can provide immediate responses, enabling features such as real-time analytics, instant notifications, and dynamic user experiences.

Helping for Microservices and Distributed Systems implementation: Event-driven architecture aligns well with the principles of microservices and distributed systems. It allows different services to communicate through events, promoting autonomy and enabling independent deployment and evolution of individual components. This architectural style appeals to organizations embracing a modular and agile approach to software development.

Event Sourcing and Event-Driven Integration: Event-driven architecture plays a crucial role in event sourcing, where events are stored as a source of truth, enabling the reconstruction of application state. Additionally, event-driven integration facilitates the exchange of data and actions across different systems and applications, making it easier to build complex, interconnected ecosystems.

Decoupled Communication and Flexibility: Event-driven architecture decouples communication between components, making it easier to introduce new functionalities or integrate third-party services without significant changes to the existing system. This flexibility allows for the evolution and adaptation of systems over time.

Support for Business and Domain-Driven Design: Event-driven architecture aligns well with business and domain-driven design approaches. It enables capturing and representing business events and domain concepts directly in the system's architecture, facilitating better understanding and alignment between the technical and business domains.

[>> Next: - EDA concepts](./eda.md)