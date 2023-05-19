# EDA with Amazon EventBridge

[My personal EventBridge summary](https://jbcodeforce.github.io/aws-studies/serverless/eventbridge).

For an EventBridge adoption in EDA, the things to consider:

* Build an inventory of Event Sources
* Address Event Schema and schema evolution and how to share them between buses
* Which consumers are interested in which events to define routing rules.
* What are the needs for exactly one delivery, ordering and time base reasoning.