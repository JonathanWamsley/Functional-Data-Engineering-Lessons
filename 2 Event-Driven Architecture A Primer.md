# Event-Driven Architecture: A Primer

By: Jake Miller

Last Updated: 10/31/18

Source: https://medium.com/high-alpha/event-driven-architecture-a-primer-f636395d0295


### Authors fasciation with school bus systems at a young age

The school bus system solved, how to get 700 students to and from thier homes and to the school builiding 182 times per year with time constraints. 

Every school day, three rows of school buses would park with engines still running. The first bell would ring and students streamed out the building boarding their bus which was in the same spot every afternoon.  

Once all the buses were full, the doors pulled shut, and a teach would signal the first driver to depart and a parade of yellow steel filled with pupils would roll up the big hill to main road where the principal would stop normal traffic so every bus could swiftly exit and deliver younglings to their home. (lol at this guys desciptive word selection)  

I'll apply this metaphor to the remaining content of this article about Event-Driven Architecture. So take note that various components of this metaphor are:  
- the principal directing traffic is a mediator
- the school busses and routes are channels
- the parking lot and stop signs are queues
- the school children are events


### Event-Driven Architecture

Several trends in computing have surfaced in recent years: Big data, containers, serverless application, microservices, and event-driven architecture (EDA). The popularity has grown because companies know they can move faster to deliver a scalable solution that are much more manageable than monolithic applications.  

What organizations are learning is that with the exponential increase in data their systems have to handle, their traditional Relational database management system (RDBMS) cannot handle the volume. Processing of information requires long running batch-based ETL to extract business insights. There is nothing wrong with patch processing of jobs for datawarehousing, but many companies are finding the need for real-time insights critical to achieving their desired outcomes.  

In the mid-2000s, N-tier architecture was at its height of popularity. Organizations would build monolithic code bases backed by RDBMS against which all CRUD was performed. Organizations are finding that the complexity of their applications has grow and their dependency graphs are virtually impossible to understand. Or, that they have piecemealed so many point-to-point integrations that they have built a gragile solution. No one can blame us engineers because, after all, object-oriented programming (OOP) encourages code reuse and, wel, at the time I really just needed service X to talk to serive Y. oops.


### A Better Approach

Organizations are starting to migrate from their monolithic to microservices architectures. Decoupling business logic and services from the event processor decreases the complexity of the architecture. Services can be developed and depoled in isolation and do not have to be aware of other services. This initial investment, in my opinion, will be paid back in spades. A microservice architecture requires a system to faciliate communication from service to service.  

By nature, an EDA facilitate a complete decentralized platform. Services don't even have to live in the same system or data center and don't have to be owned by the same organization. if a school bus system has a need for additional busses, let's say because an after-school even in a different school district, the bus system could request additional busses from one or more other systems to handle the additional load. This flexibility allows the system to expand and contract on demand. 

### Topologies

Great. You have decided that an EDA is worth the investment. You are ready to start building your new platform. Now you need to decide on the general topology of your architecture. There are two popular topologies.

### Mediator Topology

As illistrated in figure 1, the mediator topology consist of:
- events
- an event queue through which all events are processed
- a mediator which manages the order and channels in which an event is to be processed
- services which perform business logic

This is an abstract pattern so remmeber that the type of queueing techonolgy you use, and how you implement the mediator (also known as a controller or orchestrator) is up to you. We use GCP pub/sub for our queueing techonolgy and a node.js microservice running kubernetes engine and google compute engine for our controller. We then have N- number of services that process data and leverage the single-responsibility priniciple.  

![img](https://miro.medium.com/max/1430/1*qNjdvisbeEvVA7MpRzxwCQ.png)

> Applied to the bus system metaphor, imagine that a bus needs to have its brakes replaced and oil changed, and individual would beresponsible for dropping the bus off to the mechanic that will replace the brakes. Once the mechanic was done, should would inform the mediator wou would then deliver the bus to the mechanic charged with replacing the oil.

