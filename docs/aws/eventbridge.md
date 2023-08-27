# EDA with Amazon EventBridge

See all my personal [EventBridge technology summary](https://jbcodeforce.github.io/aws-studies/serverless/eventbridge).

As a serverless service, [Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide) may be used as a technology to support the event backbone capability of an EDA, leveraging the following capabilities:

- EventBus construct, to channel messages to consumers.
- EventBus pushes messages to subscribers.
- Routing rules logic to target specific consumer given message attributes, or to filter out messages.
- Add metadata to the message sent, about the source and the event type and AWS specific data like ARN, region...
- Support transforming the event before going to the target using import transformer.
- Support archiving events, and keep events for a retention period. One archive per EventBus. Events in archive can be replayed by defining a replay object, using an existing archive, and the event bus source of the archive. Replays are not immediate and can take minutes. 
- EventBridge Pipes can be used to keep message order from source to one destination, perform transformation, enrichment and filtering. The sources can be dynamoDB stream, Kinesis stream, MQ broker, MSK topic, Kafka topic, SQS queue. It processes messages in batch. 
- For error handling it supports Dead Letter Queue, with metadata about the error.

## Fit for purpose

- Events in the context of EventBridge, may not be events in the context of DDD. It is a message to route, it could mark a command as well as an event as immutable fact.
- With route and filtering logic not all messages could go to the consumer. 
- Adding a consumer, means modifying routing rules and/or filtering logic.
- One consumer receive one message.
- The EventBridge broker is responsible of the message delivery to the consumer, with timeout and retries mechanism. Events are retried for up to 24 hours or 185 times by default. 
- Resource constraints may impact the application SLA. 
- Very well integrated with other AWS services, and easy to use around Lambda function.
- EDA solution on top of EventBridge will combine more messaging systems like SNS, SQS, Kinesis data streams... 
- Delivery latency increases with more than 3 destinations and high transaction per second throughput.

## External tools to consider

* [Event Catalog](https://www.eventcatalog.dev/) is an Open Source project that helps you document your events, services and domains.
* [EventBridge Atlas](https://eventbridge-atlas.netlify.app/) an open source project to document, discover and share EventBridge schemas 

## Classical project activities

We need to consider the following project activities:

* Build an inventory of Event Sources
* Review non-functional requirements
* Define what event buses are and schema definition of the message
* Assess where to deploy event buses
* Address Event Schema and schema evolution and how to share them between buses.
* For each event bus, define the schema definition using OpenAPI, JSONSchema
* Which consumers are interested in which events to define routing rules.
* What are the requirements around exactly one delivery, message ordering and time base reasoning.
* The adoption of CloudEvent as a payload cross messaging system to keep an end to end view of the data flow.