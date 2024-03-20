# Selecting event bus technologies

**Event Backbone** serves as the core middleware that facilitates asynchronous communication and storage of events. It possesses high availability and replayability capabilities, ensuring the reliable handling and persistence of events.

As introduced in the event backbone capabilities section above, there are different messaging capabilities to support. There is no product on the market that supports all those requirements. Enterprise deployment for an event-driven architecture needs to address all those capabilities at different level, and at different time. It is important to any EDA adoption to start small, and add on top of existing foundations but always assess the best fit for purpose.

## Event Backbone with queues

Consider queue system when we need to:

* Support point-to-point delivery (asynchronous implementation of the Command pattern): Asynchronous request/reply communication: the semantic of the communication is for one component to ask a second component to do something on its data. The state of the caller depends of the answer from the called component.
* Deliver exactly-once semantic: not loosing message, and no duplication. 
* Participate into two-phase commit transaction (certain Queueing system support XA transaction).
* Keep strict message ordering. This is optional, but FIFO queues are needed in some business application.
* Scale, be resilient and always available.

Messages in queue are kept until consumer(s) got them. Consumer has the responsibility to remove the message from the queue, which means, in case of interruption, the record may be processed more than one time. So if we need to be able to replay messages, and consider timestamp as an important element of the message processing, then streaming is a better fit.

Queueing system are non-idempotents, but some are.

## Pub/sub with topics


## Streaming capabilities

Consider streaming system, like Kafka, AWS Kinesis Data Stream, as pub/sub and persistence system for:

* Publish events as immutable facts of what happened in an application.
* Get continuous visibility of the data Streams.
* Keep data once consumed, for future consumers, and for replay-ability.
* Scale horizontally the message consumption.
* When need to apply event sourcing, and time based reasoning. 

## Event router needs

Consider event router, like Amazon EventBridge when we need to apply routing to targets using business logic when

* Routing events to target based on rules
* Being able to apply filter and transformation
* Many to many routing with EventBus
* Point to point with enrichment with Pipes