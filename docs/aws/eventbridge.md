# EDA with Amazon EventBridge

[My personal EventBridge summary](https://jbcodeforce.github.io/aws-studies/serverless/eventbridge).

Using EventBridge as a technology of choice to support the event backbone of EDA, we need to consider the following project activities:

* Build an inventory of Event Sources
* The deployment of event buses
* Address Event Schema and schema evolution and how to share them between buses.
* For each event bus, define the schema definition using OpenAPI, JSONSchema
* Which consumers are interested in which events to define routing rules.
* What are the requirements around exactly one delivery, message ordering and time base reasoning.
* The adoption of CloudEvent as a payload cross messaging system to keep an end to end view of the data flow.