# Event-driven architecture

## Why EDA is important in 2020s

Since 15 years, we saw stories of different industries completely disrupted by the adoption of software solutions as a key strategic differentiator to run business. Examples in retail, movie rental, taxis, banking have demonstrated that new companies, focusing on using software, agile development, continuous deployment of new features, completely transformed legacy, brick and mortar, businesses.

This is what we could call an "industry becoming software": IT is no more a cost center or a set of computing capabilities used to run accounting, inventory, manufactoring applications... to improve a rigid business process.

So what are the key capabilities those companies used that makes them so successful. There are four major capabilities which are really defining the change to move to "business with software" at the core of the business model:

1. **Cloud**: is to rethink about data centers and optimize the compute power usage, changing the pricing model from capex to pay as you go. Cloud also provide elastic capability that developer will never be able to access in the past for adhoc use cases. 
1. **AI / machine learning** improves our way to take decisions, and automate complex tasks.
1. **Mobile** apps are redefining user experience and how we interact with businesses. Users expect a unified user interface to access the business services and get notifications when somethings interesting is happening.
1. **Data** and specially **data in motion** is very important to guide user experiences, take good decisions, automate processes, and enabling new type of applications. Data need to be available every where and in real-time, and then applications need to react when  data is created, updated...

Data architecture is evolving from a set of dedicated databases or data warehouse to distributed, decentralized architecture based on data in motion and data lake. 

The consideration of Data as a core differentiator to run a business, and as a competitive advantages, it is important that the IT architecture supports the need to get visibility of the data as soon as it created, and be able to act on it in close to real-time.

Early 2000s, the adoption of service oriented architecture help to think about business applications as a group of business services that can be ubiquitous and accessible using internet. SOAP and XML were the technologies of choice. But as early as 2004, Event-driven architecture was positioned as an evolution of SOA to scale the number of data producer or data consumer and improve inter-dependencies. The following diagram illustrates this evolution.

![](./images/soa-to-eda.png){ width=800 }

What is clear is that asynchronous communication helps in decoupling and scaling. Since mid 2010s, EDA was adopted by startup companies as a way to scale their demands, at million of users, but also to get data visibility via events. With events, it is possible to act on data as soon as created, and improve business decision automation. 

Decoupling event producers and consumers from one another, helps increasing the scalability, resilience but also the development effort.

Cloud helps enterprises' software capabilities to scale dramatically, supporting rapid change, adopting agile and continuous deployment of new features, multiple times a week to production to million of users…

 IT department should get of their 80% of time to maintain infrastructure…

There are some major drivers to adopt EDA in modern solution implementations