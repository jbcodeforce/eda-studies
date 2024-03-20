# Data lineage

When it comes to adopting a streaming data platform, there are multiple components that work together to handle data, encompassing both online and offline processing. 

In this chapter, we will explore these components and concepts that are crucial for understanding the data lineage within such a platform. To provide visual clarity, the following figure illustrates the key components involved:

![](./images/data-lineage-ctx.png)

1. **Event Sources**: Our streaming data platform receives events from various sources, including mobile applications, web single page applications, IoT devices, microservice applications, change data capture (not depicted here), and SaaS offerings. These sources generate valuable data that forms the foundation of our real-time analytics.
2. **Event Backbone:** To ensure data durability and reliability, we persist the incoming events in an append log within the event backbone. This backbone acts as a central hub, storing events with a long retention time (typically spanning multiple days). This approach allows us to maintain a complete and historical record of the data.
3. **Stream Applications:** Using streaming technologies such as Kafka Streams and Apache Flinks, our stream applications perform stateful operations to process and analyze the incoming events in real-time. These applications also enable us to perform complex event processing, providing valuable insights and actionable information.
4. **Next Best Action:** On the right side of the architecture, consumers have the ability to trigger the next best action based on specific events. This action could be a business process, some product recommendations, alerts, or any other relevant response. By leveraging the insights derived from the stream applications, we can make informed decisions and take proactive steps in real-time.
5. **Long-Term Data Storage:** While the event backbone retains data for a limited period, we often require the ability to persist data for longer durations. To achieve this, we leverage storage solutions like S3 buckets, which offer full cloud availability. Whether it's Cloud Object Storage or on-premise deployments, these storage options allow us to securely store and access data for extended periods.
6. **Big Data Processing:** To extract further value from the accumulated data, we employ big data processing platforms like Apache Spark. These platforms enable us to perform batch processing and map-reduce operations on the data at rest. By leveraging the data ingested through the event backbone, we can derive valuable insights and patterns that can drive informed decision-making.
7. **Business Dashboards:** To provide a comprehensive view of the data, our streaming data platform integrates with business dashboards. These dashboards enable users to query both static data at rest and dynamic data in motion. By utilizing interactive and streaming queries, users gain real-time access to critical information, empowering them to make data-driven decisions.


---
## Data lineage requirements

With those simple component view we can already see data lineage will be complex, so we need practices and tools to support it.
As more data is injected into the data platform, the more you need to be able to answer a set of questions like:

* Where the data is coming from 
* Who create the data
* Who own it
* Who can access those data and where
* How can we ensure data quality
* Who modify data in motion



Data lineage describes data origins, movements, characteristics, ownership and quality. As part of larger data governance initiative, it may encompass data security, access control, encryption and confidentiality. 

## Contracts

In the REST APIs, or even SOA, worlds request/response are defined via standards like OpenAPI or WSDL. In the event and streaming processing the [AsynchAPI](https://www.asyncapi.com/) is the specification to define schema and middleware binding.

### OpenAPI

We do not need to present [OpenAPI](https://www.openapis.org/) but just the fact that those APIs represent request/response communication and may be managed and integrated with the development life cycle. Modern API management platform should support their life cycle end to end but also support new specifications like GraphQL and AsynchAPI.

### AsynchAPI

Without duplicating the [specification](https://www.asyncapi.com/) is the specification to define schema and middleware binding. we want to highlight here what are the important parts to consider in the data governance:

* The Asynch API documentation which includes:
    * The server definition to address the broker binding, with URL and protocol to be used. (http, kafka, mqtt, amqp, ws)
    * The channel definition which represents how to reach the brokers. It looks like a Path definition in OpenAPI
    * The message definition which could be of any value. Apache Avro is one way to present message but it does not have to be.
    * Security to access the server via user / password, TLS certificates, API keys, OAuth2...
* The message schema

The format of the file describing the API can be Yaml or Json. 

## Schema

Schema is an important way to ensure data quality by defining a strong, but flexible, contract between producers and consumers and to understand the exposed payload in each topics. Schema definitions improve applications robustness as downstream data consumers are protected from malformed data, as only valid data will be permitted in the topic.
Schemas need to be compatible from version to version and Apache Avro supports defining default values for non existing attribute of previous versioned schema. Schema Registry enforces full compatibility when creating a new version of a schema. Full compatibility means that old data can be read with the new data schema, and new data can also be read with a previous data schema.

Here is an example of such definition:

```json
"fields": [
     { "name": "newAttribute",
       "type": "string",
       "default": "defaultValue"
     }
]
```
*Remarks: if you want backward compatibility, being able to read old messages, you need to add new fields with default value.To support forward compatibility you need to add default value to deleted field, and if you want full compatibility you need to add default value on all fields.*

Metadata about the event can be added as part of the `field` definition, and should include source application reference, may be a unique identifier created at the source application to ensure traceability end to end, and when intermediate streaming application are doing transformation on the same topic, the transformer app reference. Here is an example of such data:

```json
"fields": [
     { "name": "metadata",
       "type": {
           "name": "Metadata",
           "type": "record",
           "fields": [
               {"name": "sourceApp", "type": "string"},
               {"name": "eventUUID", "type": "string"},
               {"name": "transformApps", "type": 
                { "name" : "TransformerReference",
                  "type": "array",
                  "items": "string"
                }
               }
           ]
       }
     }
]
```


The classical integration with schema registry is presented in the figure below:

![](../../techno/avro-schemas/images/schema-registry.png)

Schema registry can be deployed in different data centers and serves multi Kafka clusters. For DR, you need to use a 'primary' server and one secondary in different data center. Both will receive schema update via DevOps pipeline. One of the main open source Schema Registry is [Apicurio](https://www.apicur.io/), which is integrated with Event Streams and in most of our implementation. Apicurio can persist schema definition inside Kafka Topic and so schema replication can also being synchronize via Mirror Maker 2 replication. If Postgresql is used for persistence then postgresql can be used for replicating schemas.

We recommend reading our 
[schema registry summary](../../techno/avro-schemas/index.md). 

Integrating schema management with code generation and devOps pipeline is addressed in this repository.


## Conclusion

Understanding the key components of a streaming data platform is essential for comprehending the data lineage within such an environment. By effectively leveraging event sources, event backbones, stream applications, long-term data storage, big data processing, and business dashboards, organizations can harness the full potential of their streaming data to drive insights, make informed decisions, and gain a competitive edge in today's data-driven landscape.