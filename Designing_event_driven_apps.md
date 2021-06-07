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
