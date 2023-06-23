# A event-driven sample solution around autonomous car ride

??? info "Updated"
    06/09/2023

The customer wants to go from one address or geographic location to another one, within a big city, using the Acme Autonomous Car Ride mobile app.

The application context looks like in the following diagram:

![](./diagrams/app-context.drawio.png){ width=600 }

Travellers use mobile application to book a ride between two locations within the same city, the Car Ride Solution dispatch an autonomous vehicle, use traffic report to compute ETA and pricing. The application is also monitoring existing rides via car telemetries. The Marketing analysis is an example of external system interested to look at the solution generated data. 

## Requirements

* Demonstrate an end to end solution with Domain Driven Design elements, focusing on an Event-Driven Architecture implementation (top down with techno mapping)
* Handle duplicate delivery from AWS EventBridge. [See proposed solution](./es-duplicate-evt.md).
* A Command Query Responsibility Segregation example
* An event-driven Saga chorerography
* A multi clusters deployment for AWS EventBridge with independant governance.

### Handle duplicate delivery with AWS EventBridge

[See proposed solution in different note.](./es-duplicate-evt.md)

### Command Query Responsibility Segregation 

CQRS is used in a lot of distributed solution, to be able to scale the read model. DynamoDB is supporting CQRS with read replicas. For more details see the [CQRS pattern explanation](../../patterns/cqrs/index.md).

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

    * Autonomous Car bounded context:

    ![](./diagrams/car-context.drawio.png)

    * Car Ride bounded context:

    ![](./diagrams/ride-context.drawio.png)

    * Customer and payment bounded contexts are not represented as we will mock them up.

## Component description

From the Domain-driven design bounded contexts, we may derive a set of microservices as illustrated in the following figure:

![](../../diagrams/classical-sync-arch.drawio.png){ width=800 }

This architecture is interesting, it embraces microservices architecture, mostly synchronous HTTP based traffic. 

* The **traveller** user is using a mobile app, connected to the classical **Backend For Frontend** service, which exposes RESTful API, with may be also a websocket connection to push notifications back to the mobile app to support traffic from backend to user.
* The major component is the **Car Ride manager** service which exposes API for the user to initiate a ride to go from a geolocation A to geolocation B, and may be an API for historical rides query.
* The **address finder**, geolocation mapper, is an utility service to map address to geo-location and any other metadata to facilitate the search for the optimal itinerary and nearest available car. It is a very important service, and may be complex to implement. It exposes HTTP APIs and must respond in sub millisecond.
* The **Car Ride** service needs to integrate with other services, like the **Payment** service once the ride is terminated, and the **Car dispatcher** to get an autonomous car.
* A **car dispatcher** needs to find the closest car to support the pickup within the shortest time. The computation may take sometime, but the response to the end user will be something like: "your car will arrive in 3 minutes and the target arrival time will be 15 minutes, do you want to proceed?". Once commited the car will move to pickup address and sends car telemetries. 
* The metrics are processed by the **route monitoring** service, which computes ETA, and other interesting real-time, time-windowing logic.
* When the travel is completed, the **payment** service needs to trigger the payment and the reward program service may update the number of travel, and may be also rate the consumer. As there is no driver, there is no more driver rating. 

## Adopting an event-driven approach to the implementation

The fact that we discovered event during the DDD phase does not mean we adopt EDA. Other non functional requirements need to be considered, like scalability, contract coupling, streaming data to integrate as part of creating derived business event that are important to the business process. As introduced in the SOA to EDA section, a business process model can be done to understand the flow of command / data and even events. 

Let revisit the business process flow in more detail using the commands, aggregates and events we discovered during DDD: 

![](./diagrams/bpm-flow.drawio.png){ width=1000 }

