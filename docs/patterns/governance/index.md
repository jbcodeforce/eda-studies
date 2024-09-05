# Event driven solution governance

Digital businesses are characterized by real-time responsiveness, scalability, and flexibility, all while focusing on delivering an outstanding customer experience. The adoption of APIs and microservices has played a significant role in enabling this transformation, supporting real-time interactions and increasing levels of agility through the decoupling of applications. 

However, digital business requires more than just these capabilities. It needs to become more time-sensitive, contextual, and event-driven in nature. Events are the fundamental building blocks of how modern digital businesses operate, and an event-driven architecture allows IT to align with and support this way of working.

By emphasizing events, digital businesses can better respond to changing circumstances, anticipate customer needs, and adapt their operations accordingly. This event-driven approach helps organizations become more nimble, responsive, and customer-centric – key attributes for success in the digital age.

Governing the development, deployment, and maintenance of an event-driven solution involves considering a variety of viewpoints. In this section, we'll cover what we believe are the important factors to address


The first crucial question to answer is: **do we truly need to use an event-driven solution?** This is all about ensuring the chosen approach is a good fit for the specific requirements and challenges at hand.

Most event-driven architecture (EDA) adoption starts from the adoption of a new programming model based on the reactive manifesto, or by selecting a modern middleware platform like Kafka to support loosely coupled microservices. However, this is typically done within the context of a single business application.

To effectively govern an event-driven solution, organizations need to take a broader, enterprise-wide perspective. They must consider the overall strategic alignment, technical feasibility, and organizational readiness before committing to an EDA approach.


After the initial successful implementation of an event-driven solution, it's important to consider adopting a more strategic and enterprise-wide approach. This includes:

* **Technology Selection**: Establishing a standardized set of technologies for event-driven architecture, rather than ad-hoc choices for each project.
* **Common Architecture and Design Patterns**: Defining and consistently applying architectural and design patterns to ensure scalability, maintainability, and reusability of the EDA.
* **Data Governance and Lineage**: Implementing robust data governance practices to control data models and maintain comprehensive data lineage across the event-driven ecosystem.
* **Streamlined Onboarding and Deployment**: Adopting common methodologies and processes to quickly onboard new business initiatives and deploy new capabilities on top of the EDA, ideally in a matter of days or weeks.


## Fit for Purpose

In the context of event-driven architecture (EDA), the 'fit for purpose' assessment needs to help answer several high-level questions:

1. **When should we use an event-driven solution for a new application?** This involves evaluating the specific requirements, challenges, and benefits that an event-driven approach can address.
1. **When should we use a modern pub/sub event backbone versus traditional queuing products?** [See this dedicated note on comparing those messaging approaches](../../concepts/fit_for_purpose.md/#)
1. **What data stream processing capabilities are required?** Determining the appropriate data stream processing tools and techniques is crucial for deriving insights and responding to events in a timely manner.
1. **What are the different use cases and benefits of event-driven architecture?** Clearly articulating the various applications and advantages of EDA helps justify its adoption and guide the implementation strategy.


## Architecture patterns

We have already described in [this chapter](../../concepts/eda.md#components-of-the-architecture) as set of event-driven architecture patterns that can be leveraged while establishing EDA practices which includes how to integrate with data in topic for doing feature engineering. 

Legacy integration and coexistence between legacy applications or mainframe transactional application and microservices is presented in [this section](../../concepts/legacy-itg.md).

The [modern data pipeline](../../concepts/data-pipeline.md) is also an important architecture pattern where data ingestion layer is the same as the microservice integration middleware and provide buffering capability as well as stateful data stream processing.

From an implementation point of view the following design patterns are often part of the EDA adoption:

* [Strangler pattern](../index.md/#strangler-pattern)
* [Event sourcing](../event-sourcing/index.md)
* [Choreography](../index.md/#choreography)
* [orchestration](../index.md/#orchestration)
* [Command Query Responsibility Segregation](../cqrs/index.md)
* [Saga pattern](../saga/index.md)
* [Transactional outbox](../index.md/#transactional-outbox)
* [Event reprocessing with dead letter](../dlq/index.md)


---

TBC

## Getting developers on board

## Data lineage

Data governance is a well established practices in most companies. Solutions need to address the following needs:

* Which sources/producers are feeding data?
* What is the data schema?
* How to classify data
* Which process reads the data and how is it transformed?
* When was the data last updated?
* Which apps/ users access private data?

In the context of event-driven architecture, one focus will be to ensure re-use of event topics, 
control the schema definition, have a consistent and governed ways to manage topic as service, 
domain and event data model definition, access control, encryption and traceability. 
As this subject is very important, we have started to address it in a [separate 'data lineage' note](../../patterns/data-lineage/index.md).

## Deployment Strategy

Hybrid cloud, Containerized


security and access control
Strimzi - cruise control
active - active
topic as self service

## Ongoing Maintenance

Procedures performed regularly to keep a system healthy
Cluster rebalancing
 Add new broker - partition reassignment

Product migration


## Operational Monitoring 

assess partition in heavy load
assess broker in heavy load
assess partition leadership assignment

Problem identification
System health confirmation

## Error Handling

Procedures, tools and tactics to resolve problems when they arise