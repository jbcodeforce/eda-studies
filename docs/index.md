# Introduction

???- Info "Site updated 09/17/2023"
    September:
    - microservice challenges in patterns chapter
    - flow architecture summary
    - Dead letter queue pattern

    August:
    
    - Reactive systems
    - EDA content for event backbone criteria

    July: 
    
    - Add Choreography and orchestration introduction in design pattern
    - Add Scatter-gather in design pattern
    - Add Kafka connect content
    - Add event sourcing 
    - Tune Saga pattern content

## Why this site?

IT architects have a lot of initiatives around cloud adoption, and refactoring existing applications to distributed microservices as the values are no more to proof those days, but more the how to do it. So this web site is like a global book on Event-Driven Architecture adoption. Instead of writing a book, that is static by design, I prefer to do this web site so I can adapt content and add new chapters while I learn new things in this exciting domain.

## Target audience

Overall, Event-Driven Architecture offers numerous benefits, including improved scalability, performance, loose coupling, real-time analysis capabilities, support for microservices, event sourcing, and flexible integrations. These advantages make it an attractive option for audiences interested in building modern, responsive, and scalable applications.

I see that Enterprise Architects, Solution Architects, and developer leaders will be interested in this content.

## What kind of problems are we trying to solve?

In contemporary times, the term Event-driven architecture is eavily influenced by the capabilities of software vendors. While this site does include some implementations using specific products, its primary aim is to remain product-agnostic and to outline the essential principles of EDA.

When moving to cloud native implementation architects need to address:

* scalability and Performance,
* the Microservice and distributed system, adoption and well designed with good granularity,
* coupling of services with point to point synchronous communication brings some implementation challenges and frictions for changes,
* the support for real-time processing to act on the data as early as data created, 
* considering events as a source of truth to get eventual consistent systems, be more resilient, 
* how to get started and applied proved methodology.

## EDA key benefits

For Scalability and Performance, Event-driven architecture empowers applications to seamlessly scale and efficiently handle heavy workloads. By facilitating asynchronous processing, it allows components to respond to events as they unfold, resulting in enhanced performance and responsiveness.

For loose coupling: an Event-driven architecture fosters a decoupled relationship among various system components by facilitating communication through events. This approach minimizes inter-component dependencies, facilitating smoother maintenance, updates, and scalability. Such flexibility is highly attractive to developers and architects who prioritize modularity and extensibility.

Event-driven architecture excels in constructing real-time and reactive systems, as it enables applications to respond to events in real-time, thereby facilitating immediate reactions and empowering functionalities such as real-time analytics, instant notifications, and dynamic user experiences.

Event-driven architecture seamlessly aligns with the tenets of microservices and distributed systems, offering a communication framework that facilitates service interaction via events. This fosters autonomy, enabling independent deployment and evolution of individual components, making it an attractive choice for organizations committed to a modular and agile software development approach

EDA assumes a pivotal role in both event sourcing, where events serve as a definitive record for reconstructing application state, and event-driven integration, which streamlines data and action exchange across diverse systems and applications, simplifying the development of intricate, interconnected ecosystems

Event-driven architecture facilitates decoupled communication between components, making it easier to introduce new functionalities or third-party services with minimal impact on the existing systems. This inherent flexibility supports the ongoing evolution and adaptation of systems as needed.

Support for Business and Domain-Driven Design: Event-driven architecture aligns well with business and domain-driven design approaches. It enables capturing and representing business events and domain concepts directly in the system's architecture, facilitating better understanding and alignment between the technical and business domains.

[>> Next: - EDA concepts](./eda.md)