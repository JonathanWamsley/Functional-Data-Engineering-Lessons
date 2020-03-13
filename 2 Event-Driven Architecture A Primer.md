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

### Broker Topology

In broker topology, as illustrated in Figure 2, rather than leaverage a centralized mediator for for orchrestrating event and services, services subscribe to channels, execute their business logic, and then publish a new message to which other services subscribe. An advantage of this approach is that be removing the need for a mediator you have reduced complexity. The disadvantage is that coordination and enforcement of execution order are not handled. Again, the pattern is agnostic to technolgy.  

![fig 2](https://miro.medium.com/max/1427/1*EfgHczAOIpZXjtRs5yhvbQ.png)

> Applied to the same school bus system metaphor, if a bus has to have it's brakes repaced and then the oil changed, in a broker topology, the bus would be dropped off to the mechanic replacing the brakes. Once the job was done, he'd put it in the parking lot where the oil-changing technician would know to look and pull into the garage to change the oil. Irealize that example is a stretch but you get the point. 

### Performance

In EDA, there are two important factors for performance, throughput and latency. The greater the latency, the lower the throughput. You ahve two options to continue to improve performance of your system: decrease latency by optimizing code or configurations, or increase throughput by adding additional resources.  

When it comes to measuring performance, I recommend that your team define Service Level Objectives (SLO) and Service Level Indicators (SLI) for each service in your platform. We have adopted this approach and it allows us not only to monitor and gauge our success in production, but it gives us a guidepost for analzying benchmark and performance test results prior to launching new features.  

If you have not read it, read Site Reliability Engineering by Google  

If there is one thing i have learned from building scalable platforms, it is that every milisecond matters. 1ms does not matter at low volume but add 1ms to the processing of 1 million messages and you have added 15 minutes to processing time. Adding 1ms for one billion messages adds 277 hours.  

> Every milisecond matters. Adding 1 milisecond of processing to one billion messages add 277 hours to processing time.  

Here are some recommednations based on experience:  
- be smart about what data you include on your payloads
- add constraints to the amount of data you incldue on your payload to control performance and expense
- never use transactional database as a queue, especially when you expect high volumes.
    - Databases like SQL have to write every transaction to a transaction log. Waits on writes to the transaction log will kill your performance.
    - Also, reading and writing from the same table will 100% result in locks on the table and IO waits that drastically increase IO wait times.
- there are two dates that matter in event-driven systems:
    - the actual even data
        - the time at which a user or system action occured
    - the proccessed date
        - usually the time the even was ingested by the system
    - the distinction in date is important because your architecture should manage late arrivals if you are performing any sort of logic within a window
    
> Given the last bullet point, imagine that the school principal is required to count all the students that arrive to school for that day. She spends 20 minutes as all the busses arrive and ends up with 620 students. Then she stops counting. The window closed. But let's say that one bus was 1- minutes late because a traffic light was out of service. The principal would have to go back and add the additional 80 students to her original tally. This same sort of reconciliation would have to occur in your system.  


### Sledgehammer Syndrome

Finally, event-driven architecture is not the right pattern for all applications. Quite the contrary, such a pattern introduces the complexity of its own.

> Don't fall victim to Sledgehammer Syndrom

EDA could very likely be a sledgehammer for your problem when you instead need a screwdriver. Take into account the cost of development and maintainance when deciding if this is the right solution for you.  

When considering the complexity that the solution will require and whether it's worth the investment, determine how you will tackle the following:  

When considering the complexity that the solution will require and whether it is worth the investment, determine how you will tackle the following:  

1. what is the right level of fidelity of service abstraction
2. are your event messages schema or schema-less
3. how does your system handle failures caused by bad or corrupt data, downed services or queues
4. how do you handle a noisy neighbor problem (sharing cloud resources with other services affect your performance if they are hogging bandwidth)
5. how will you debug and undestand the flow of events through the system
6. how do you handle events that are not idempotent
7. how will your system handle cycle prevention and detection
8. how will your system handle rollback of a distrubuted transaction

### Conclusion

To sum it up, EDAs are:  
- highly scalable and decentralized
- can processes events and perform functions like aggregate at time of ingestion
- eliminate point-to-point integration

Consider emplying an EDA when:  
- you require low latency and high volume processing
- you require aggregating or processing data real-time within a window
- more than one service needs to processes the same event
- you need to horizontally scale a distributed system
- you want to implement a microservice architecture

