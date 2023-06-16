# A event-driven sample solution around autonomous car ride

??? info "Updated"
    06/09/2023

The customer wants to go from one address or geographic location to another one, within a big city, using the Acme Autonomous Car Ride mobile app.

The high-level component architecture may look like in the following figure:

![](../../diagrams/classical-sync-arch.drawio.png){ width=800 }

This architecture is interesting, it embraces microservices architecture, mostly synchronous HTTP based traffic. 

## Requirements

* Demonstrate an end to end solution with Domain Driven Design elements, focusing on an Event-Driven Architecture implementation (top down with techno mapping)
* Handle duplicate delivery from AWS EventBridge
* A Command Query Responsibility Segregation example
* An event-driven Saga chorerography
* A multi clusters deployment for AWS EventBridge with independant governance.

### Handle duplicate delivery with AWS EventBridge

The problem we will try to demonstrate how to handle duplicate records. 
This is a common pattern in any messaging system, where producer or consumer retries can generate multiple messages.

Producer to EventBridge may generate duplicate messages while retrying to send a message because of communication issue or not receiving acknowledgement response. The producer code needs to take into account connection failure, manage retries.

The SDK for EventBridge includes a method called [put_events](https://docs.aws.amazon.com/eventbridge/latest/APIReference/API_PutEvents.html) to send 1 to many events to a given EventBus URL. The records sent are ecapsulated with an envelop with, at the high level, the following structure:

```json
{ "Source": "reference of the producer",
  "Resources": "aws ARN"
  "DetailType": "CarRideEventType",
  "Time": datetime.today().strftime('%Y-%m-%d'),
  "Detail": data,
  "EventBusName": targetBus
}
```

The returned response includes the an event Id for each entry sent, so it may be possible that some of the event failed processing, and in this case there will be an error code.

Only the consumer of the message can identify duplicate record. EventBridge is not a technology where the broker support a protocol to avoid duplication, as Kafka does. 

So with EventBridge we need to assign a unique identifier to each message, and a counter and track the processed messages using this identifier and the current count. 

Here is an example of payload creation in python:

```python
def defineEvent(data):
    attributes = {
        "type": "acme.acr.CarRiderCreatedEvent",
        "source": "acr.com.car-ride-simulator",
        "eventId": str(uuid.uuid1()),
        "eventTime": datetime.today().strftime('%Y-%m-%d %H:%M:%S.%f'),
        "eventCount": 1
    }
    eventToSend = { "attributes": attributes, "data": data}
    print(eventToSend)
    entries = [
                { "Source": attributes["source"],
                "DetailType": "CarRideEvent",
                "Time": datetime.today().strftime('%Y-%m-%d'),
                "Detail": json.dumps(eventToSend),
                "EventBusName": targetBus,
                "TraceHeader": "carRideProcessing"
                },
            ]
    return entries

```

At the consumer side before processing a new message, check if its identifier already exists in the system. If it does, consider it a duplicate and discard it. Consider the attributes:

```json
    "source": "acr.com.car-ride-simulator",
    "eventId": str(uuid.uuid1()),
    "eventCount": 1
```

We can use a time window horizon to keep the last n seconds messages to limit the search. 

This is important when the semantic of the event is about creating data. If the producer generates a OrderCreated event, then de-duplication may be important. As an alternate processing is to support idempotency: in dual CreatedEvent, if the count > 1 then and the record with the same identifier is already in the backend, discard any new record, but if the message is UpdateEvent then update existing record (like a database will do).


The implementation of this solution leverages, [CloudEvents](https://cloudevents.io), a well not established event structure to share metadata among technology agnostic component. 

A EventBridge'sresponse Entries array can include both successful and unsuccessful entries. As for each record, the index of the response element is the same as the index in the request array, it is possible to identify the message in error. 

There are some error due to the EventBridge service, like a ThrottlingException or InternalFailure that may be retried. Some should never be retried but sent to a DLD queue or saved in a temporary storage for future automatic processing or for manual processing. 

Example of code to handle errors:

```
```

To simplest minimum demonstration the solution, we may use the following components:

![](./diagrams/exactly-once-eb.drawio.png){ width=1000 }

A producer is exposing and API to support POST on CarRides resources to request for a new car ride. The application persists the data in a backend like DynamoDB and sends an event like `CarRideCreated` or `CarRideUpdated` event to an customer event bus for downstream distribution. In EDA the very important element is the fact that an event can be valuable to any consumer, so adding a SNS Topic to implement a fan-out distribution will make sense.
Finally ordering of events is important, so the consumer should be getting message from a Fifo Queue.


Another may be more elegant implementation is to use the outbox pattern, and write the events to a table in DynamoDB, do change data capture on this outbox table, using DynamoDB Streams, use EventBridge pipe to process the streaming data and the send to targets, which could be a SNS.

![](./diagrams/exactly-once-eb-dynamo.drawio.png){ width=1000 }

Other considerations:

Message Time-To-Live (TTL): Set a Time-To-Live (TTL) value for messages in the queue. If a message remains in the queue beyond its TTL, consider it expired and discard it. This helps prevent the processing of stale or duplicate messages that might have been delayed or requeued due to failures.


### Command Query Responsibility Segregation 

CQRS is used in a lot of distributed solution, to be able to scale the read model. DynamoDB is supporting CQRS with read replicas. For more details see the [CQRS pattern explanation]().

### Saga pattern

The classical implementation of Saga is to use an orchestrator to manage the state of the Saga and being able to rollback the transaction with compensation API. For more details see the [Saga pattern explanation]().


## Domain-driven design applied

### Event Storming

We will mock an event storming exercise which generates the following elements:

* Discovered Events from a process point of view. Mostly happy path

    ![](./diagrams/events-ddd.drawio.png)

* Event Reorganized by concerns: Rides, Autonomous Car, Payment, Award

    ![](./diagrams/events-concern-ddd.drawio.png)

### DDD

* Aggregates: represent the main business entity within the domain and sub-domain

    ![](./diagrams/aggregate-ddd.drawio.png)

* Domain/Sub-domains

    ![](./diagrams/domain-ddd.drawio.png)

* Commands

    ![](./diagrams/command-ddd.drawio.png)

* Bounded Contexts 

    * Autonomous Car

    ![](./diagrams/car-context.drawio.png)

    * Ride Context

    ![](./diagrams/ride-context.drawio.png)

## Component description

From the Domain-driven design bounded context we can build a set of microservices. 

* The **traveller** user is using a mobile app, connected to the classical **Backend For Frontend** service, which exposes RESTful API, with may be also a websocket connection to push notifications back to the mobile app to support traffic from backend to user.
* The major component is the **Car Ride manager** service which exposes API for the user to initiate a ride to go from a geolocation A to geolocation B, and may be an API for historical rides query.
* The **address finder**, geolocation mapper, is an utility service to map address to geo-location and any other metadata to facilitate the search for the optimal itinerary and nearest available car. It is a very important service, and may be complex to implement. It exposes HTTP APIs and must respond in sub millisecond.
* The Car Ride service needs to integrate with other services, like the Payment service once the ride is terminated, and car dispatcher to get an autonomous car.
* A car dispatcher needs to find the closest car to support the pickup within the shortest time. The computation may take sometime, but the response to the end user will be something like: "your car will arrive in 3 minutes and the target arrival time will be 15 minutes, do you want to proceed?". Once commited the car will move to pickup address and sends car telemetries. 
* The metrics are processed by the route monitoring service, which computes ETA, and other interesting real-time, time-windowing logic.
* When the travel is completed, the payment service needs to trigger the payment and the reward program service may update the number of travel, and may be also rate the consumer. As there is no driver, there is no more driver rating. 