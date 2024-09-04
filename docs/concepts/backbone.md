# Messaging Backbone

As introduced before the messaging as a service enterprise may want to deploy support the following components:

![](./diagrams/event-backbone.drawio.png)

Most of those components, withn this diagram, are described in separate chapters. 

**Message Backbone** serves as the core middleware that facilitates asynchronous communication and storage of events. It possesses high availability and replayability capabilities, ensuring the reliable handling and persistence of events.

## Selecting event bus technologies

As introduced in the [event backbone capabilities section](eda.md/#zooming-into-the-capabilities-of-the-event-backbone), there are different messaging capabilities to support. There is no product on the market that supports all those requirements. Enterprise deployment for an event-driven architecture needs to address all those capabilities at different level, and at different time. It is important to any EDA adoption to start small, and add on top of existing foundations but always assess the best fit for purpose.

### Message Backbone with queues

Consider queue system when we need to:

* Support **point-to-point** delivery (asynchronous implementation of the **Command** pattern): Asynchronous request/reply communication: the semantic of the communication is for one component to ask a second component to do something on its data. The state of the caller depends of the answer from the called component.
* Deliver **exactly-once** semantic: not loosing message, and no duplication. 
* Participate into **two-phase commit transaction** (certain Queueing system support XA transaction).
* Keep strict **message ordering**. This is optional, but FIFO queues are needed in many business applications.
* Scale, be resilient and always available.

Messages in queue are kept until consumer(s) got them. Consumer has the responsibility to remove the message from the queue, which means, in case of interruption, the record may be processed more than one time. So if wdevlopers need to be able to replay messages, and consider timestamp as an important element of the message processing, then streaming is a better fit.

Queueing system are non-idempotents, but some are.

**Products:** [IBM MQ](../techno/ibm-mq/index.md), open source RabbitMQ and Active MQ. Some Cloud vendor managed services like AWS SQS, Azure Storage Queues.

![](../techno/ibm-mq/diagrams/mq-topologies.drawio.png)

**IBM MQ Broker Cluster topologies**

### Pub/sub with topics

The main event backend on the market is Apache Kafka. Kafka's append-only log format, sequential I/O access, and zero copying support high throughput with low latency. Its partition-based data distribution allows it to scale horizontally to hundreds of thousands of partitions. Older queue systems are also supporting pub/sub, most of them based on the Java Messaging Service APIs and protocol.

A typical kafka cluster has multiple brokers, with their own persistence on disk and Zookeeper ensemble to manage the cluster states.

![](../techno/kafka/diagrams/kafka-hl-view.drawio.png)

Topics are a key elements to support message persistence and access semantic. Most of the systems work with topic subscriptions, and some will published on the active subscriptions only, while Kafka help to have consumer being able to connect at any time and replay the event from the origin.

### Criteria to select a technology

A Message as a Service platform will need to support queues and topics. For Topics one of the Kafka flavor needs to be selected. The list of Kafka packaging is long and Cloud providers offer managed services to by pass the management of the infrastructure and version to version upgrade.

There are a set of criteria to consider for selecting a Kafka packaging:

* Type of asynchronous expected interaction
* Data volume
* Easy to move data to 3nd tier storage
* Latency
* Storage
* User interface to get visibility to topic, consumer groups and lags
* Support to declarative configuration
* Monitoring with proprietary and shared platforms
* Access control: enforce strong access controls and authentication mechanisms
* Encryption for data in transit and at rest
* Support to a single glass for queue and topic management
* Legal and compliance requirements for data handling, with fine grained access control
* Cost of running the infrastructure and services.
* Support to address complex disaster recovery strategies

### Centralized Cluster vs multiple clusters

Adopting a centralized Kafka cluster, for production environment, can bring numerous benefits, but it also requires careful consideration of various criteria. it is important to consider the segregation strategies for maximizing performance and optimizing cost while increasing Kafka adoption. The strategic dimensions to consider are:

* Operational decoupling
* Tenant isolation
* Application optimization
* Service availability, goelocation, and data criticality
* Cost and operating attention to each workload.

#### Factor for centralized

 Here are some key factors to consider:

* Number of applications and number of topic per application
* Application designed for real-time messaging and distributed data processing may better fit for leveraging common cluster. While ETL data pipeline can afford more downtime than a real-time messaging infrastructure for frontline applications.
* Data volume and data criticality
* Data isolation by applications and domains: a centralized Kafka cluster facilitates better data governance, data sharing, and data consistency across the organization. It enables a common data streaming strategy and reduces the risk of data silos. But domain specific data boundaries leads to adopt decentralized cluster.
* Retention time
* Network latency for producers and consumers connecting to the centralized cluster.
* Streaming processing of consumer-process-produce with exactly once semantic can only be done in the same cluster.
* Scalability needs in the future: data volume growth
* Maximun number of broker per clusters- Cluster with 100 brokers is alaready complex to manage. A small cluster needs at least 3 to 6 brokers and 3 to 5 Zookeeper node. Those components should be spread across multiple availability zones for redundancy.
* Level of automation to manage the cluster: a centralized approach simplifies management, and maintenance. It reduces the complexity of configuration, upgrades, and troubleshooting compared to managing multiple independent clusters
* Monitoring of the cluster health with a way to drill down to the metrics for root cause analysis
* Current expertise and resources to manage a centralized system effectively.
* Legal and compliance requirements for data handling.
* Cost with bigger hardware for each node, number of nodes, versus smaller cluster. Consider idle time. Centralized cluster should cost less thant multiple decentralized clusters. By consolidating resources and reducing redundant infrastructure and DevOps efforts, organizations can achieve significant cost savings. But if the data transfer to the centralized Kafka cluster is high then the benefits will be low.
* Evaluate how well a centralized cluster can integrate with existing systems and data sources.
* Plan for how the centralized system will handle failures and ensure data durability. Bigger failover is more complex and may be more costly.
* Replication Strategies: Assess the replication needs to prevent data loss in case of cluster issues.
* Determine if a centralized cluster promotes better collaboration among teams.
* Consider how operational responsibilities will be divided among teams.
* Kafka was not designed for multi-tenancy. Ther is no concept like namespaces for enforcing quotas and access control. Confluent has added a lot on top of the open-source platform to support.

*Sharing a single Kafka cluster across multiple teams and different use cases requires precise application and cluster configuration, a rigorous governance process, standard naming conventions, and best practices for preventing abuse of the shared resources.*

#### Adopting multiple clusters

The larger a cluster gets, the longer it can take to upgrade and expand the cluster due to rolling restarts, data replication, and rebalancing. Using separate Kafka clusters allows faster upgrades and more control over the time and the sequence of rolling out a change.

* There is nothing in Kafka for a bad tenant to monopolize the cluster resources, having smaller cluster for each functional area or domain, is better for isolation.

#### Practices

* Adopt a topic naming convention with prefixes becomes part of the security policy to be controlled with access control list.
* Limit admin access to one user per domains, clarify the rule of operations.
* There may be need for hundreds of granular ACLs for some applications that are less trusted, and coarse-grained ACLs for others.
* Add a way to manage metadata about cluster, topics, schema, producer and consumer apps.

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