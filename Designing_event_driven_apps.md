# Designing Event-driven Applications Using Apache Kafka Ecosystem

## Event-driven Architecture (EDA)

A software architecture pattern promoting the production, detection, consumption of, and reaction to events.  
When number of microservices increase to solve a business use case, it increases the communication between the services which results to something called ***microservices hell***. That is when event-driven architecture comes into the picture. Everything revolves around events and not the Data.
- **Event driven microservices** architecture is a great solution to many problems.  
- **Serverless** applications and **cloud computing services** (function as a service: FaaS) can solve problems that require small, short lived applications.  
- **Streaming** model is used to respond fast to customer's needs (companies react in almost real time, since its an endless loop of events arising from the customers).
- **Event-sourcing**: Storing the data as a sequence of events and re-creating the current state based on the log.
- **CQRS**: Command Query Responisibility Segregation works by separating the interfaces for read and write. Tradaitionally, both reading and writing data are done using the same set of entities. Using CQRS, the interfaces are separated and exposed over different APIs.
![Screenshot 2021-06-07 at 4 41 52 PM](https://user-images.githubusercontent.com/10058009/121007101-4c67b000-c7af-11eb-8531-84900c0e79ce.png)

### Messages, Events and Commands

These 3 form very important pieces of any event driven architecture. Their meanings should be pretty clear.

### Benefits of EDA

- **Decoupling**: When service A needs to communicate with service B, it uses a Broker based technology, by trasmitting the message to the broker and broker decides where the message needs to be sent. Service A only needs to know the location of the broker. Direct communication b/w service A and B is forbidden since it results in direct coupling.
- **Encapsulation**: Having clean boundaries of each event, without any confusion.
- **Optimization**: EDA is designed to run in real time.
- **Scalability**: Easy to horizontally scale the application to handle more requests.

### Drawbacks

- Steep learning curve
- Complex to maintain: What happens if something goes wrong? What happens in case of duplicate events?
- Loss of transactionality: It becomes difficult to revert if something fails, since an event might trigger lot of small changes.
- Lineage: Events can become lost or corrupted, and because of decoupling its difficult to identify the source of such events. Solution: Adding an identifier to the event telling which all applications the event has passed through.

### Event Storming

In a typical system, the system design revolves around the data, but in case of event driven, it revolves around event (you dont care how your data looks like). The approach to design EDA is called **Event Storming** combined with **Domain-driven design**.

Event Storming is used to model a whole business line with domain events as a result of a collaboration of different of the organization.

#### The Workshop:
- Things needed
  - Need a room for different people to participate. No chairs, since its interactive session
  - Invite the right people. A facilitator (one of knows about the workshop) decides the minimum number of person for the meeting. No upper limit. Couple of architects, developer, UX and a domain expert (someone who knows how the business works and its processes) are required.
  - Unlimited modeling space: to work out the process. A huge paper/empty wall can be used.
  - All events will be written on sticky notes of different colors. There is a color Scheme of the sticky notes:
    - Domain events (something that is happening in the system and is relevant for the business): orange. Usually the domain event may provoke an action or even other events that affect the system.
    - Policy: purple. Refers to a process occured by an event, and it always starts with a keyword, "Whenever". Example: Whenever an account is created, we send an email confirmation.
    - External System: pink. Refers to any interaction that happens with an external system and you do not have control over it. Example: External payment provider like paypal.
    - Commands: blue. Refers to the actions initiated by a user or system like a scheduled job. Different between command and event is that, command resides at the beginning of an event flow, triggering the event chain.
  - Food: since the workshop may take a whole day or maybe even more in case of complex use cases.
- Once we have all this, we move to the DDD (Domain-driven design) step. To model the software, we need to identiy the aggregate by logically grouping various commands and events together. Goal is to define structures that are isolating related concerns from one another.
- Next we define the bounded context: Allowing the use of same terms in different subdomains. Example: "Received" in the order system is different compared to "Received" in the shipping system.

#### Example: User checkout process (Shopping)

Step 1: Event Storming. 

![Screenshot 2021-06-07 at 9 13 31 PM](https://user-images.githubusercontent.com/10058009/121049378-45539880-c7d5-11eb-9c40-d905e6b185d5.png)

Step 2: DDD. 

![Screenshot 2021-06-07 at 9 15 32 PM](https://user-images.githubusercontent.com/10058009/121049691-92376f00-c7d5-11eb-88de-49faa3e8c27d.png)

### Why Kafka?

- Open Source
- Written in Java (originally written in Scala, but the bytecode can also be run on a JVM)
- High throughput:
  - because there is not serialization or deserialization happening inside kafka. What kafka receives and transmits is only bytes.
  - zero copy: when the message is received, in a typical system the network card copies it to JVM heap which puts it into hard drive. But in case of Kafka, JVM heap does not come into picture. (only available for non-TLS connections, because TLS protocol is deeply embedded in the JDK, hence its not possible in such situations)
- More than a messaging system: Its a distributed streaming platform (messaging system, distributed storage with fault tolerance, data processing: process events as they occur). By using streaming, all incoming events can be processed in almost real time.

### Kafka Producer Record

![Screenshot 2021-06-08 at 2 19 26 PM](https://user-images.githubusercontent.com/10058009/121154453-9eb6d880-c864-11eb-99d4-2fa0a37d4101.png)
