# EDA adoption assessment question

This section lists a set of questions architects can use to assess the EDA deployment and adoption.

## Definitions used in the questions

| Term | Definition |
| --- | --- | 
|**Event** | represents a significant change in state or an occurrence that has happened within a system. It is a domain-specific occurrence that reflects a change in the state of the domain model |
|**Event backbone** |	core middleware that facilitates asynchronous communication and storage of events. It possesses high availability and replayability capabilities, ensuring the reliable handling and persistence of events. |
|**Event schema** |	structured definition that describes the format and structure of an event within an event-driven system. Avro, Json, XSD are common way to define schemas |
|**Queue** |	Queues are addressable locations to deliver messages to and store them reliably until they need to be consumed. | 
|**Topics** | represent end-points to publish and consume records | 
|**Multi-tenant** |	a single instance of software serves multiple customers or tenants|
|**Domain** | 	we use the domain driven design definition :  specific area of knowledge, activity, or interest that is the focus of a particular application |
|**Bounded context** |	boundaries within which a particular domain model applies, ensuring clarity and focus. |

## Business

| Question | Response | Assessment |
| --- | --- | --- |
| What specific business goals are you aiming to achieve with event-driven architecture? | | |
| How will this architecture align with your overall digital transformation strategy? | | |
| Is the EDA is a tactical strategy to address some application requirements around resilience and scaling ? or is it more strategic initiative? | | |
| Do you use event in few application, isolated, mostly supplementary to other development activities? | | |
| Are you taking any business decision, automatically, on the events created in close real-time? | | |
| Do you have a strategic use and governance of EDA, promoted cross-organization by IT leadership? | | |
| Do you have some governance authority for API, Events, business services, business process and data definitions? A center of excellence, or competence. | | |
| Do you want to move to managed services and scale to zero infrastructure? | | |
| Do you adopt a multi cloud provider strategy? | | |

## Current architecture

| Question | Response | Assessment |
| --- | --- | --- |
| Do you use an event-bus currently?  | | |
| Do you use a shared event cluster between domains?  | | |
| Do you use queues in point to point and topics for pub/sub or both?  | | |
| Which technology for queueing and messaging?  | | |
| How many event backbone clusters?  | | |
| How many node per cluster in average?  | | |
| Do you use baremetal nodes or VM based or pure container on k8s?  | | |
| Do you deploy cluster in different regions / Datacenters?  | | |
| Do you consider datalake or lakehouse to be a target sink for events?  | | |
| How many applications are using events / messaging today?  | | |
| Does some of those topics / queues exposed as part of a B2B interaction?  | | |
| Do you classify your business applications into different business impact classification ? if so are all the critical applications using asynchronous communication today?  | | |
| If classifications are in place can you share uptime requirements?  | | |
| What are the DR needs today?  | | |
| Are you using process orchestration product today? | | |
| Do you have remote applications that needs access data with lower latency? how many of them? | | |
| Do you have exposed topic or queue to the public internet? | | |
| How many application are exposing business services today? REST or SOAP? | | |
| Do you use schema registry? How many of them? | | |
| Do you use Avro schema, protobuf or JSON or a combination of them? None is possible too.  | | |
| Do you use AsyncAPI document to document topics and schema?  | | |
| Do you generate technical events from file upload to bucket notification and content management?  | | |
| Do you consider logs to be a source of events?  | | |
| Do you use container and orchestration like Kubernetes in your modern application development practices?  | | |
| How long do you need to keep events in event backbone? | | |
| For audit reason do you need to keep message for longer term ? Do you need to replay event of the past? | | |
| Do you use some dedicated connector framework to integrate systems to your event backbone?  | | |
| Do you use topic replication between event clusters today? | | |
| Do you use stateful processing on the events to compute aggregate within time windows? | | |
| Are you using an API management platfor; today? Does it support AsyncAPI management too? | | |

## Future architecture

| Question | Response | Assessment |
| --- | --- | --- |
| How many applications will use events / messaging next year?  | | |
| How many applications need to be changed to generate events or consume events?  | | |
| Can you give us the scalability requirements in term of transaction per second on average and at peak?  | | |
| Do you need transactional applications with strong consistency be integrated in your EDA?  | | |
| Do you need to integrate data coming from mainframe?  | | |
| Do you plan to use streaming for consuming event, processing them and generate new events?  | | |
| Do you have a need to manage topic from a single user interface?  | | |
| Do you want to apply event-sourcing as an implementation pattern?  | | |

### Event sources

| Question | Response | Assessment |
| --- | --- | --- |
| What are you connecting in term of native applications ? | | |
| Do you use change data capture today to get event from database? | | |
| What are data replications ? | | |
| Where are existing data sources ? | | |
| Is there any SaaS integration for data source? | | |
| Do we have requirements for private networking? | | |

### Event Sinks

| Question | Response | Assessment |
| --- | --- | --- |
| Do you plan to use Lakehouse technology to persist event?  | | |
| Do you plan to use database to persist data from event? | | |
| Where are you connecting to? | | | 


## DevOps

| Question | Response | Assessment |
| --- | --- | --- |
| How many developers implement asynchronous event generation for business applications? | | |
| Are schema management part of your DevOps practices? | | |
| Do you automate event backbone cluster creation with Infrastructure as Code? | | |
| Do you automate topic and queue creation? | | |
| Do you have the necessary in-house skills, or will you need to hire or train staff? | | |
| Do you use Domain-driven design and event storming to model the application logic with events? | | |
| Do you have automatic process to sccale up the messaging middleware? | | |
| Do you manage consumer lag behind for the event processing? | | |

## Governance

| Question | Response | Assessment |
| --- | --- | --- |
| Does the organization has standard practices?  | | |
| Are the API and Event designs and development coordinated?  | | |
| Do you have a process to identify message consumers automatically?  | | |
| Do you track data lineage today from producer to different consumers and to sinks?  | | |


## Security

| Question | Response | Assessment |
| --- | --- | --- |
| What security measures are needed to protect event data? | | |
| Do you need to uniquely control access to topic or queue for each application? | | |