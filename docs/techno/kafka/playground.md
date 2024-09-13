# Kafka development sandbox

## Quarkus dev

If any Kafka-related extension is present (e.g. quarkus-smallrye-reactive-messaging-kafka), Dev Services for Kafka automatically starts a Kafka broker in dev mode and when running tests.

* [Using the REST Client](https://quarkus.io/version/3.2/guides/rest-client-reactive)
* [Apache Kafka Reference Guide](https://quarkus.io/version/3.2/guides/kafka) with dev service
* [Testing Your Application](https://quarkus.io/version/3.2/guides/getting-started-testing)

## Kafka local

When using docker compose in the [eda-quickstart]() repository, the compose starts one zookeeper and one Kafka broker locally using last Strimzi packaging, one Apicurio for schema registry and Kafdrop for UI.

In the docker compose the Kafka defines two listeners, for internal communication using the DNS name `kafka` on port 29092 and one listener for external communication on port 9092.

![](./images/docker-kafka.png)

A container in the same network needs to access kafka via its hostname. 

To start [kafkacat](https://hub.docker.com/r/edenhill/kafkacat) and [kafkacat doc to access sample consumer - producer](https://github.com/edenhill/kafkacat#examples)

```shell
docker run -it --network=host edenhill/kafkacat -b kafka:9092 -L
```