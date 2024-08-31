# Confluent Cloud

SaaS offering for the Confluent platform on one of AWS, Azure or GCP cloud.

Below is a set of important information, facts, links on the offering.

## Kakfa cluster

* One environment can have multiple clusters
* To access one API key/secret pair for each Kafka cluster

## Schema Registry

* One registry per environment, created in the same region as the first kafka cluster is created, but can manage future clusters in other regions.
* In multi-tenant deployments, one physical Schema Registry per cloud and geographic region, hosts many logical schema registries. Confluent Cloud uses API keys that are resource scoped for Schema Registry clusters to store schemas and route requests to the appropriate logical clusters.
* This is different API key than the one to access the kafka cluster
* At the environment level, developers may view and search schemas, monitor usage, and set a compatibility mode for schemas. 
* Still compatibility mode can be overrided anged at the topic/schema level. The default is backward compatibility: consumer can consume older message, as default values are added to new attributes.
* *Foward* compatibility means that data produced with a new schema can be read by consumers using the last schema, or the schema before. [See details in compatibility note](https://docs.confluent.io/cloud/current/sr/fundamentals/schema-evolution.html#summary). With *Foward* compatibility mode, consumers aren’t guaranteed to be able to read old messages.
* When apps are in VPC, the VPC needs to be able to communicate with  Confluent Cloud public end point on port 443
* Stream governance feature helps addressing data governance
* Schema can be created via CLI, REST API, Console or Maven plugin to be used during CI/CD
* Schema is associated to topic, with mechanism to support backward compatibility
* Avro was developed with schema evolution in mind, and its specification clearly states the rules for backward compatibility, not the case for Json or Protobuf.
* Transitive compatibility means checking a new schema against all previously registered schemas.
