# Event-driven architecure getting started

## Project approach

A common question I got in enterprise architecture workshop and event-driven architecture establishments is how to start. So here are a simple list of things that can be done for a given project:

1. Start by [Event Storming workshop](../methodology/event-storming/index.md) with the line of business subject-matter expert to model the process from an  event point of view
1. Define the domain, sub-domains, commands, aggregates, events and bounded contexts using Domain-driven design.
1. For a given bounded context, assess the life cycle of the business entity and defines events for state change.
1. Externalize the event schema definition in Avro and start adopting automatic construction of POJO or Python classes from schema and then pipeline to push to repository. Any integration needs to leverage schema.
1. In Java thin to use reactive messaging and implement consumer code that manage read-offset commitment

## Demonstration code

### Pub/sub with Kafka, Quarkus reactive messaging and Avro

[EDA quickstart - Java Avro First Demo](https://github.com/jbcodeforce/eda-quickstarts/tree/main/java-avro-first-demo): The demonstration illustrates the following concepts:

* ava applications with Quarkus framework and Microprofile 3.0 reactive messaging to produce and consumer order event messages
* Avro schema definition, own by the order manager component producer of events
* App uses health, metrics and OpenAPI extensions
* Use Apicurio schema registry and the maven plugin to upload new definition to the connected registry.

### Python producer / consumer with Avro and Kafka