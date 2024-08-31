# Apache Avro, Data Schemas and Schema Registry

???- Info Updates"
  Created 01/2019  Last update 6/20/2022

This chapter describes what and why Avro and Schema registry are important elements of any event-driven solutions.

## Why this is important

Loosely coupling and asynchronous communication between applications does not imply the absence of a contract to enforce certain constraints between producers and consumers.

When we refer to a contract, we can initially think of schemas, as we did with XSD. In the context of JSON, JSON Schema and Avro schemas can be utilized to define the data structure of messages. Given the need for metadata in messaging, [CloudEvents](https://cloudevents.io) has gained widespread acceptance as a specification for describing event data. Additionally, [AsyncAPI](../../patterns/api-mgt/#support-for-async-api) establishes standards for events and messaging in the asynchronous landscape from an API perspective. It combines message schemas, channels, and binding definitions, providing consumers with the essential information needed to access a data stream or queue.

The contract is defined by a schema. From an event-driven architecture (EDA) design perspective, the producer is responsible for defining the schema, as it oversees the life cycle of the main business entities from which business events are generated. The producer ensures that the message complies with the schema for serialization.

In addition to these specifications, various technologies support contract management, including schema registries and API managers. The following
figure presents the foundations for integration between producers, schema registry and consumers.

![schema registry management](./diagrams/schema-registry.drawio.png)

The Schema Registry offers producer and consumer APIs that allow them to determine whether the event they are about to produce or consume is compatible with previous versions or aligns with the expected version. To achieve this, both producers and consumers need access to the schema definition during serialization and deserialization.

These can be done either by:

1. Reading the schema from a local resource to the producer application using a file, a variable, a property (or a kubernetes construct such as a configmap or a secret).
2. Retrieving the schema definition from the Schema Registry given a name/ID.

When the producer wants to send an event to a Kafka topic, two things happen:

1. The producer makes sure the event to be sent complies to the schema. Otherwise, it errors out.
2. Determine  whether the event they are about to produce or consume is compatible with previous versions or compatible with the version they are expecting.

For the first action, the producer already has what it needs: the schema definition and the event that must comply with it. This enables the producer to perform the compliance check. However, for the second action, where the producer must ensure that the schema definition of the event is compatible with the existing schema for the relevant topic (if one exists), the producer may need to consult the Schema Registry.

Producers (and consumers) maintain a local cache of schema definitions (and their versions) along with their unique global IDs, all retrieved from the Schema Registry, for the topics they wish to produce or consume events from. If the producer has a schema definition in its local cache that matches the schema definition required for the event's serialization, it can simply send the event, as the definitions match and the event complies with the schema.

However, if the producer does not have a matching schema definition in its local cache—whether due to different versions or simply lacking the definition—it will contact the Schema Registry.


If a schema definition for the relevant topic that matches the schema definition required for serialization already exists in the Schema Registry, the producer can simply retrieve the global unique ID for that schema definition. It will then locally cache both the schema definition and its ID for future events, thereby avoiding the need to contact the Schema Registry again.

If a schema definition for the relevant topic that matches the schema required for serialization does not already exist in the Schema Registry, the producer must register the schema definition it has on hand for that topic. This ensures that the schema is available for any consumer wishing to consume events that comply with it.

To achieve this, the producer must be configured for auto-registration of schemas and provided with the appropriate credentials to register schema definitions, especially if the Schema Registry implements any form of role-based access control (RBAC).

If the producer is not configured to auto-register schema definitions or if its credentials do not permit registration, the send event action will fail. This will persist until a schema definition for the relevant topic, matching the schema required for serialization, is registered in the Schema Registry.

If the producer is configured to auto-register schema definitions and has the necessary credentials, the Schema Registry validates the compatibility of the current schema definition required for serialization with any existing schema definitions for the relevant topic. This ensures that if the schema definition being registered is a newer version, it remains compatible, preventing any negative impact on consumers.

Once the schema definition or the new version of an existing schema is successfully registered in the Schema Registry, the producer retrieves its global unique ID to keep its local cache up to date.


!!! Warning
    We strongly recommend that producers be restricted from registering schema definitions in the Schema Registry to ensure better governance and management. It is advisable that schema definition registration be handled by a designated individual with the appropriate role, such as an administrator, API manager, asynchronous API manager, development manager, or operator.


Once the producer has a schema definition in its local cache, along with its global unique ID, that matches the schema required for the serialization of the event, it produces the event to the Kafka topic using the appropriate `AvroKafkaSerializer` class. The global unique ID of the schema definition is serialized alongside the event, eliminating the need for the event to carry its schema definition, as was the case in older messaging or eventing technologies and systems.

By default, when a consumer reads an event, the schema definition for that event is retrieved from the Schema Registry by the deserializer using the global unique ID specified in the consumed event. The schema definition is retrieved from the Schema Registry only once, when an event contains a global unique ID that is not found in the consumer's local schema definition cache.

The global unique ID can be located either in the event headers or within the event payload, depending on the producer application's configuration. When the global unique ID is located in the event payload, the data format begins with a magic byte, which serves as a signal to consumers, followed by the global unique ID and the message data as usual. For example:

```
# ...
[MAGIC_BYTE]
[GLOBAL_UNIQUE_ID]
[MESSAGE DATA]
```

Be aware that non-Java consumers may use a C library that also requires the schema definition for the deserializer, as opposed to allowing the deserializer to retrieve the schema definition from the Schema Registry, as explained above. The strategy should involve loading the schema from the Schema Registry via an API. See [Kafka python reported issue](https://github.com/confluentinc/confluent-kafka-python/issues/834)

## Schema Registry

### Apicurio

[Apicur.io](https://www.apicur.io) includes a [schema registry](https://www.apicur.io/registry/docs/apicurio-registry/2.6.x/index.html) to store schema definitions. It supports Avro, Json, protobuf schemas, and an API registry to manage OpenApi and AsynchAPI.

Apicur.io is a Cloud-native Quarkus Java runtime with low memory footprint and fast deployment times. It supports [different persistences](hhttps://www.apicur.io/registry/docs/apicurio-registry/2.6.x/getting-started/assembly-installing-registry-docker.html)
like Kafka, Postgresql, Infinispan and supports different deployment models.

#### Registry Characteristics

* Apicurio Registry is a datastore for sharing standard event schemas and API designs across API and event-driven architectures.
* The registry supports adding, removing, and updating the following types of artifacts: OpenAPI, AsyncAPI, GraphQL, Apache Avro, Google protocol buffers, JSON Schema, Kafka Connect schema, WSDL, XML Schema (XSD).
* Schema can be created via Web Console, core REST API or Maven plugin
* It includes configurable rules to control the validity and compatibility.
* Client applications can dynamically push or pull the latest schema updates to or from Apicurio Registry at runtime.
Apicurio is compatible with existing Confluent schema registry client applications.
* It includes client serializers/deserializers (Serdes) to validate Kafka and other message types at runtime.
* Operator-based installation of Apicurio Registry on OpenShift
* Use the concept of artifact group to collect schema and APIs logically related.
* Support search for artifacts by label, name, group, and description

When using Kafka as persistence, special Kafka topic `<kafkastore.topic>` (default `_schemas`), with a single partition, is used as a highly available write ahead log. 
All schemas, subject/version and ID metadata, and compatibility settings are appended as messages to this log. 
A Schema Registry instance therefore both produces and consumes messages under the `_schemas` topic. 
It produces messages to the log when, for example, new schemas are registered under a subject, or when updates to 
compatibility settings are registered. Schema Registry consumes from the `_schemas` log in a background thread, and updates its local 
caches on consumption of each new `_schemas` message to reflect the newly added schema or compatibility setting. 
Updating local state from the Kafka log in this manner ensures durability, ordering, and easy recoverability.


The way Apicur.io has to handle schema association to topics is by schema name. Given we have a topic called orders, 
the schemas that will apply to it are avros-key (when using composite key) and orders-value (most likely based on CloudEvents and then custom payload).

## Apache Avro

Avro is an open source data serialization system that helps with data exchange between systems, programming languages, and processing frameworks. Avro helps define a binary format for your data, as well as map it to the programming language of your choice.

### Why Apache Avro

There are several websites that discuss the Apache Avro data serialization system benefits over other messaging data protocols. A simple google search will list dozens of them. Here, we will highlight just a few from a [Confluent blog post](https://www.confluent.io/blog/avro-kafka-data/):

- It has a direct mapping to and from JSON
- It has a very compact format. The bulk of JSON, repeating every field name with every single record, is what makes JSON inefficient for high-volume usage.
- It is very fast.
- It has great bindings for a wide variety of programming languages so you can generate Java objects that make working with event data easier, but it does not require code generation so tools can be written generically for any data stream.
- It has a rich, extensible schema language defined in pure JSON
- It has the best notion of compatibility for evolving your data over time.

## Data Schemas

Avro relies on schemas. When Avro data is produced or read, the Avro schema for such piece of data is always present. This permits each datum to be written with no per-value overheads, making serialization both fast and small. An Avro schema defines the structure of the Avro data format.

### How does a data schema look like?

Let's see how a data schema to define a person's profile in a bank could look like:

```json
{
  "namespace": "banking.schemas.demo",
  "name": "profile",
  "type": "record",
  "doc": "Data schema to represent a profile for a banking entity",
  "fields ": [
    {
      "name": "name",
      "type": "string"
    },
    {
      "name": "surname",
      "type": "string"
    },
    {
      "name": "age",
      "type": "int"
    },
    {
      "name": "account",
      "type": "banking.schemas.demo.account"
    },
    {
      "name": "gender",
      "type": {
        "type": "enum",
        "name": "genderEnum",
        "symbols": [
          "male",
          "female"
        ]
      }
    }
  ]
}
```

???- "Notice:"
    1. There are primitive data types like `string` and `int` but also complex types like `record` or `enum`.
    2. Complex type `record` requires a `name` attribute but it also can go along with a `namespace` attribute which is a JSON string that qualifies the name.
    3. Data schemas can be **nested** as you can see for the `account` data attribute. See below.

```json
{
  "namespace": "banking.schemas.demo",
  "name": "account",
  "type": "record",
  "doc": "Data schema to represent a customer account with the credit cards associated to it",
  "fields": [
    {
      "name": "id",
      "type": "string"
    },
    {
      "name": "savings",
      "type": "long"
    },
    {
      "name": "cards",
      "type": {
        "type": "array",
        "items": "int"
      }
    }
  ]
}
```

In the picture below we see two messages, one complies with the above Apache Avro data schema and the other does not:

![data examples](./images/data_examples.png)

You might start realising by now the benefits of having the data flowing into your Apache Kafka event backbone validated against a schema. See next section for more.

For more information on the Apache Avro Data Schema specification see <https://avro.apache.org/docs/current/spec.html>

### Benefits of using Data Schemas

- **Clarity and Semantics**: They document the usage of the event and the meaning of each field in the "doc" fields.
- **Robustness**: They protect downstream data consumers from malformed  data, as only valid data will be permitted in the topic. They let the producers or consumers of data streams know the right fields are need in an event and what type each field is (contract for microservices).
- **Compatibility**: model and handle change in data format.

## Avro, Kafka and Schema Registry

In this section we try to put all the pieces together for the common flow of sending and receiving messages through an event backbone 
such as kafka having those messages serialized using the Apache Avro data serialization system and complying with their respective messages 
that are stored and managed by a schema registry.

Avro relies on schemas. When Avro data is produced or read, the Avro schema for such piece of data is always present. An Avro schema defines 
the structure of the Avro data format. Schema Registry defines a scope in which schemas can evolve, and that scope is the subject. 
The name of the subject depends on the configured subject name strategy, which by default is set to derive subject name from topic name.

In this case, the messages are serialized using Avro and sent to a kafka topic. Each message is a key-value pair. Either the message key or the message value, or both, can be serialized as Avro. Integration with Schema Registry means that Kafka messages do not need to be written with the entire Avro schema. Instead, Kafka messages are written with the **schema id**. The producers writing the messages and the consumers reading the messages must be using the same Schema Registry to get the same mapping between a schema and schema id.

## More reading

### Articles and product documentation

* [Apicur.io schema registry documentation](https://www.apicur.io/registry/docs/apicurio-registry/2.1.x/index.html)
* [Confluent schema registry overview](https://docs.confluent.io/platform/current/schema-registry/index.html)
* [Producer code with reactive messaging and apicurio schema registry](https://github.com/jbcodeforce/eda-quickstarts/tree/main/quarkus-reactive-kafka-producer)
* [Consumer code with reactive messaging and apicurio schema registry](https://github.com/jbcodeforce/eda-quickstarts/tree/main/quarkus-reactive-kafka-consumer)

