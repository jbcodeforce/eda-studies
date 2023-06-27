# Handle duplicate delivery with AWS EventBridge

**Problem statement**: how to handle duplicate records when using AWS EventBridge as event backbone?. 

This is a common problem in many messaging systems, where producer retries can generates the same message multiple times or consumer retries on the subscribed topic, may generate duplicate processing of the received message.

The following diagram presents the potential problem. The left lambda function exposes an API to create "CarRide" and send message to Event Bus that the CarRide was created. The same applies if the carRide is updated, like for example, the customer accepts the proposed deal so a car can be dispatched:

![](./diagrams/eb-duplicate-problem.drawio.png){ width=700}

EventBridge is not a technology where the broker supports a protocol to avoid duplication, like Kafka does with the idempotence configuration for producer. So producer may generate duplicate. As we will see in producer section below, we can add event id specific to the application to trace duplicate records end to end.The only things that can be done in producer code is to send messages not processed by the event bus to a Dead Letter Queue (may be after some retries). 

Remember in a pure EDA point of view, consumer can be added over time to support new use cases, producer should not be aware of those consumers, they just publish events as their internal state changes. 

Amazon [EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguid) has the following important characteristics that we can leverage to address de-duplication:

* Routing rules to filter or fan-out to a limited set of targets.
* Message Persistence with possible replays.
* Event replications to event bus in another region.
* Pipes to do point to point integration between source and destination.

### Focus on producer processing

Producer to EventBridge may generate duplicate messages while retrying to send a message because of communication issue or not receiving an acknowledgement response. The producer code needs to take into account connection failure, and manage retries.

The SDK for EventBridge includes a method called [put_events](https://docs.aws.amazon.com/eventbridge/latest/APIReference/API_PutEvents.html) to send 1 to many events to a given EventBus URL. Each record sent includes an envelop with the following structure:

```json
"Entries": [
    { 
    "Source": "reference of the producer",
    "Resources": "aws ARN of the producer app"
    "DetailType": "CarRideEventType",
    "Time": datetime.today().strftime('%Y-%m-%d'),
    "Detail": data,
    "EventBusName": targetBus
    }
]
```

The message is, in fact, including more parameters as a set of [common parameters](https://docs.aws.amazon.com/eventbridge/latest/APIReference/CommonParameters.html) are defined for each different EventBridge API mostly for versioning and security token. 

A EventBridge's response Entries array can include both successful and unsuccessful entries.

The returned response includes an event `Id` (created by EventBridge) for each entry sent processed successfully, or an error object. As for each record, the index of the response element is the same as the index in the request array, it is possible to identify the message in error and resend it. 

There are some error due to the EventBridge service, like a ThrottlingException or InternalFailure that may be retried. Some should never be retried but sent to a DLD queue or saved in a temporary storage for future automatic processing or for manual processing. 

To better support an EDA approach, we need to assign a unique identifier to each message, with a sequencer and track the processed messages using this identifier and the current count. We should not leverage the eventID created by the event bus, as it is local to this bus, and we may need to go over other components or even another event bus in another region.

Here is an example of payload creation in python, with the eventId (the one from the producer point of view and not the EventBridge created one) as part of the `attributes` element (the metadata part of [CloudEvents.io](https://cloudevents.io)): 

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

This approach support the event replication use case, when deploying in two regions, and use EventBridge Global Endpoint:

![](./diagrams/eb-duplicate-2-regions-problem.drawio.png)

If a failover is triggered by Route 53 health check error on the primary event bus, then messages are sent to secondary region, and duplicates can be managed the same way as in a mono-region. Except if dynamoDB is not accessible by itself from the first region. In this case Global Table may be used. 

For EventBridge configuration here is an example of producer configuration in a SAM template:

```yaml
Resources:
  CarRideEventBus:
    Type: AWS::Events::EventBus
    Properties:
      Name: !Ref EventBusName
 CarRideSimulatorFunction:
    Type: AWS::Serverless::Function 
    Properties:
      Environment:
        Variables:
          EventBus: !Ref EventBusName 
    Policies:
      - Statement:
        - Sid: SendEvents
          Effect: Allow
          Action:
          - events:PutEvents
          Resource:
            - !GetAtt CarRideEventBus.Arn
```

### The Consumer part

Only the consumer of the message can identify duplicate records. If the consumer is not using an idempotent backend for persistence, then it needs to track the messages that it has processed in order to detect and discard duplicates. 

We assume the consumer is looking at the events, from another bounded context, and it is interested to subscribe to the event bus to support its own joining queries for example or participate in a long-running transaction. If the consumer uses DynamoDB as persistence layer then it can use the `[updateItem](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateItem.html)` API to edit the existing event's attributes, or adds a new event to the table if the event does not already exist.  

![](./diagrams/consumer.drawio.png)

If for example the consumer manages RideTracking, it can keep event about CarRide entity creation, and then use a dedicated table to keep the CarRide with the eventId as key. Duplication is just an overwrite. A putItem may also work, and an exception about duplicate key will do nothing.

If we do not want to update content, when processing a message, a consumer can detect and discard duplicates by querying the database on the index and cancel the received message if eventID key exist.

The DynamoDB configuration is mono-region multi AZ for high availability but single writer to make it more simple. 

If we do not use DynamoDB, we can use simple cache. With an application running continuously, single instance we can use HashMap. But when we define multiple concurrent instances of the application, like lambda function, we need to use a distributed, clustered data cache.

At the consumer side before processing a new message, the code checks if the eventID already exists in the system. If it does, consider it a duplicate and discard it. From the metadata sent by the producer consider the following attributes:

```json
    "source": "acr.com.car-ride-simulator",
    "eventId": str(uuid.uuid1()),
    "eventCount": 1
```

We can use a time window horizon to keep the last n seconds messages to limit the search, and evict data in the cache. 

This is important when the semantic of the event is about creating data. If the producer generates a OrderCreated event, then de-duplication is important. When the count > 1 for the same identifier, discard any new record.

The implementation of this solution leverages, [CloudEvents](https://cloudevents.io), a well not established event structure to share metadata among different technology stacks. 

The code implementation use AWS MemoryDB for Redis to get a distributed cache on a multi-AZs deployment:

```python

```

The problem is to evict older events from the cache. A scheduled processing may be used to remove any message older to current timestamp and n minutes. The n can be computed by assessing the risk to get a duplicate messages within this time window, and the size of the expected memory.

--- 

Example of code to handle errors:

```
git clone https://github.com/aws-samples/serverless-patterns/ 
cd serverless-patterns/lambda-eventbridge-sns-sam
```

To simplest minimum demonstration the solution, we may use the following components:

![](./diagrams/exactly-once-eb.drawio.png){ width=1000 }

A producer is exposing and API to support POST on CarRides resources to request for a new car ride. The application persists the data in a backend like DynamoDB and sends an event like `CarRideCreated` or `CarRideUpdated` event to an customer event bus for downstream distribution. In EDA the very important element is the fact that an event can be valuable to any consumer, so adding a SNS Topic to implement a fan-out distribution will make sense.
Finally ordering of events is very important to build an accurate view of the data, so the consumer should be getting messages from a Fifo Queue to keep order.


Another may be more elegant implementation is to use the outbox pattern, and write the events to a table in DynamoDB, do change data capture on this outbox table, using DynamoDB Streams, use EventBridge pipe to process the streaming data and the send to targets, which could be a SNS.

![](./diagrams/exactly-once-eb-dynamo.drawio.png){ width=1000 }

Other considerations:

Message Time-To-Live (TTL): Set a Time-To-Live (TTL) value for messages in the queue. If a message remains in the queue beyond its TTL, consider it expired and discard it. This helps prevent the processing of stale or duplicate messages that might have been delayed or requeued due to failures.