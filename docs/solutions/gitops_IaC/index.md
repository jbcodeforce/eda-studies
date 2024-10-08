# Kafka and Event-Driven deployment with GitOps and Infrastructure as Code 

???- info "Updates"
    Created 08/2024 from older notes

The goal of this chapter is to go over the deployment considerations for a cloud-native event-driven application using GitOps / Infrastructure as Code practices.

The core concept of GitOps is to maintain a single Git repository that consistently holds declarative descriptions of the desired infrastructure in the production environment. An automated process ensures that the production environment aligns with the state described in the repository.

## Context

In this chapter, we want to demonstrate the following components deployment: 

![](../../techno/avro-schemas/diagrams/schema-registry.drawio.png)

1. Kafka cluster
1. Schema registry
1. Topic definitions
1. Producer and Consumer Apps
1. Avro schema deployment

Which is an instantiation of the broader EDA blueprint as illustrated in the following diagram:

![](../../concepts/diagrams/eda-hl.drawio.png)

To better improve DevOps automation and the overall solution governance, **GitOps** and **Intrastructure as Code** practices should help deploying all those components as source code.

We will also address some generic development practices for schema management.

--- 

## Components for GitOps

The following diagram illustrates the technical components involved in a typical production deployment of an event-driven solution.

![components](./diagrams/components.drawio.png){ width="1000" }

The diagram organizes components according to when they are introduced during system development—either early or late—and whether they are high-level application-oriented components or low-level system-oriented components. For example, GitHub serves as a foundational system component essential for structuring the deployment of event-driven solutions. In contrast, streaming or event-driven applications are higher-level components that depend on the prior deployment of other components.

The color coding indicates that blue components are part of the solution, red components are associated with GitOps on Kubernetes or RedHat OpenShift, and green components are external to the Kubernetes cluster, even if they could potentially be integrated. [Helm](https://helm.sh) and [Kustomize.io](https://kustomize.io/) representsa way to define deployments, while Sealed Secrets is a service for managing secrets.

???- definition "Helm & Helm Chart"
    **Helm** is a package manager for Kubernetes that simplifies the deployment and management of applications on Kubernetes clusters. **Helm Chart** is a collection of files that describe a related set of Kubernetes resources. It includes a Chart.yaml file containing metadata about the chart, templates for the Kubernetes manifests, and a values.yaml file for configuration settings. Helm charts allow users to package applications and share them easily, facilitating consistent deployment and versioning.

Among the later components to deploy, we have included everything necessary to monitor both the solution and the infrastructure.

### Event-driven applications

Event-driven applications support business logic, are based on microservices, and utilize reactive messaging through message queues (MQ) or Kafka APIs. These applications offer OpenAPIs for mobile and web applications and provide AsyncAPI specifications for producing events to Kafka or messages to MQ. Both OpenAPI and AsyncAPI definitions are managed by the API manager and the event endpoint manager.

![](./diagrams/eda-dev-govern.drawio.png)

Schema definitions are managed by a **Schema Registry**.

### Event-streaming applications

Event-streaming applications support stateful processing using [Kafka Stream](../../techno/kstreams/index.md) APIs or [Apache Flink](https://jbcodeforce.github.io/flink-studies/).

### Kafka Cluster

When using Cloud provider, managed service for Kafka, the cluster can be defined using infrastructure as code. When deploying Kafka onto Kubernetes platform [Strimzi](https://strimzi.io/) defines customer resources with operator to declare any Kafka components as Yaml manifests. See the [Strimzi getting started](https://strimzi.io/quickstarts/) to install the Strimzi Operator on a local kubernetes. Once the operator is running it will watch for new custom resources and create the Kafka cluster, topics or users that correspond to those custom resources.

### Queue manager

A queue manager provides queuing services through various MQ APIs. It hosts the queues that store messages produced and consumed by connected applications and systems. Queue managers can be interconnected via network channels, enabling messages to flow between disparate systems and applications across different platforms, including both on-premises and cloud environments.


---

some note on labs. See deployment under eda-rt-inventory solution

```sh
kubectl get pod -n kafka --watch
kubectl logs deployment/strimzi-cluster-operator -n kafka -f
```