# EDA with Amazon EventBridge

[My personal EventBridge summary](https://jbcodeforce.github.io/aws-studies/serverless/eventbridge).

[Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide) may be used as a technology to support the event backbone capability of an EDA leveraging the following capabilities:

- EventBus to channel message to consumers
- Supporting Routing rules logic to target specific consumer given message attributes. Filtering out messages.
- Add metadata to the message sent, about the source and the event type and AWS specific data like ARN, region.
- Supports transforming the event before going to the target using import transformer
- Supports archiving events, and keep events for a retention period. One archive per EventBus. Events in archive can be replayed by defining a replay object, using an existing archive, and the event bus source of the archive. Replays are not immediate and can take minutes. 
- EventBridge Pipes can be used to keep message order from source to one destination, perform transformation, enrichment and filtering. The sources can be dynamoDB stream, Kinesis stream, MQ broker, MSK stream, Kafka topic, SQS queue. It processes messages in batch. 

Keep in mind:

- events in the context of EventBridge may not be events in the context of DDD. It is a message to route, it could mark a command as well as an event.

We need to consider the following project activities:

* Build an inventory of Event Sources
* The deployment of event buses
* Address Event Schema and schema evolution and how to share them between buses.
* For each event bus, define the schema definition using OpenAPI, JSONSchema
* Which consumers are interested in which events to define routing rules.
* What are the requirements around exactly one delivery, message ordering and time base reasoning.
* The adoption of CloudEvent as a payload cross messaging system to keep an end to end view of the data flow.