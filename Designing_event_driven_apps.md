# Designing Event-driven Applications Using Apache Kafka Ecosystem

## 2. Event-driven Architecture (EDA)

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

## 3. Why Kafka?

- Open Source
- Written in Java (originally written in Scala, but the bytecode can also be run on a JVM)
- High throughput:
  - because there is no serialization or deserialization happening inside kafka. What kafka receives and transmits is only bytes.
  - zero copy: when the message is received, in a typical system the network card copies it to JVM heap which puts it into hard drive. But in case of Kafka, JVM heap does not come into picture. (only available for non-TLS connections, because TLS protocol is deeply embedded in the JDK, hence its not possible in such situations)
- More than a messaging system: Its a distributed streaming platform (messaging system, distributed storage with fault tolerance, data processing: process events as they occur). By using streaming, all incoming events can be processed in almost real time.

### Kafka Producer

![Screenshot 2021-06-08 at 2 19 26 PM](https://user-images.githubusercontent.com/10058009/121154453-9eb6d880-c864-11eb-99d4-2fa0a37d4101.png)

### Kafka Consumer

![Screenshot 2021-06-08 at 2 34 58 PM](https://user-images.githubusercontent.com/10058009/121156943-c4dd7800-c866-11eb-948d-e8eb66df1c23.png)

## 4. Communicating Message Structure with AVRO and Schema Registry

**Serialization:** The process of translating data structures or object states into a format that can be stored, transmitted and reconstructed later, possible in a different computer environment

There can be different serialization formats, the most common of which is Binary Serialization (more compact and thus, its faster when transmitted). Drawback is that data is not human readable during the transfer. **Schemas** enforce a strict data structure. Some data serialization format allow flexible structure while others enforce using a specific one using a schema.

### Popular Serialization Formats:

- JSON: Uses text serialization; and there is no schema involved.
- XML: Uses well known text serialization; Schema (not mandatory) can be used to enforce a structure.
- YAML: Uses text serialization; No schema involved
- Avro: Developed as part of Apache Hadoop; Uses binary serialization; Uses JSON-based schemas to define a structure.
- Protobuf: Also known as protocol buffers uses binary schema; developed by Google and offers a simple and performant way of storing and interchanging data within systems; Uses interface description language to define structures.
- Thrift: developed by Facebook for scalable cross language services development; Uses binary format; Uses interface description language to define structures.
<img width="899" alt="Screenshot 2021-06-10 at 9 24 17 PM" src="https://user-images.githubusercontent.com/10058009/121557278-3f0e2800-ca32-11eb-8e54-367152f2cd44.png">

### Avro Serialization Format:

Offers a rich data structure which can be stored within container files. Applications use Avro for remote procedure calls by using a simple integration with dynamic languages like Groovy, JavaScript, Python. It also offers code generation and improved performance in statically typed languages like C#, Java. Since its a binary serialization format, the data is compressed in a compact format making it lighter compared to JSON, XML serialization formats. Avro uses JSON based schemas to define data structures. These schemas are either embedded in container files or transferred as separate objects. File extension for Avro schema is avsc, but the content of a JSON format.

Schema has the following details:
- type: It can be either a prmitive or a complex type.
  - primitive types:
    - null: no value
    - boolean
    - int: 32 bit signed integer
    - long: 64 bit signed integer
    - float: 32 bit floating point
    - double: 64 bit floating point
    - bytes: sequence of bits
    - string: unicode character sequence
  - complex types:
    - record: combination of multiple fields 
    - enum: predefined list of values (`{"symbols": ["BLUE", "GREEN"]}`)
    - array: to store a list of values
    - maps: to store key value type of data. keys are always string (`{"key": "value"}`)
    - unions: used when you have optional values (`["null", "string"]`)
    - fixed: used when you need to store precise number of bytes.
- namespace
- name of the schema: together with the namespace, it defines the full schema name
- fields: Declared fields contained by the record. Each field record can have a special attribute as well. Below `dateOfBirth` represents no. of days from the Epoch date.
```json
{
  "type": "record",
  "namespace": "com.pluralsight",
  "name": "User",
  "fields": [
    {
      "name": "userId",
      "type": "string"
    },
    {
      "name": "username",
      "type": "string"
    },
    {
      "name": "dateOfBirth",
      "type": "int",
      "logicalType": "date"
    }
  ]
}
```

#### Avro Serialization/Deserialization

- Initially, we have a user data and a Schema that are passed to a serializer.
- Using the passed schema, the user data is converted to a binary object, which can be stored on a hard drive or transferred across a network. 
- To get back the user data, we use a deserializer. Without the user schema, the deserializer cannot convert the binary object back to the user data. Even with a slightly changed schema, the deserialization will fail.
- After deserialization, the user object can be used for further processing.

#### Generating Java Class from Avro Schema:

- Need `avro tools` to generate the class from the avsc file. Use wget to download it.
- Command to generate classes: `java -jar <name of jar> compile schema <path of the schema file> <directory where you want class files>`
- The generated class is more verbose than usual because it uses lot of methods for performance optimization.

#### Schema Registry

Application that handles the distribution of schemas to producers and consumers and stores them for long-term availability. Schema registry stores this information using a kafka topic. 

In order to receive the right schema, a proper mechanism needs to be in place. **Subject name strategy** achieves that by categorizing the schemas based on the topic they belong to. The subject name for key will be `{topic-name}-key`: `user-tracking-key`. Subject name for value will be `{topic-name}-value`: `user-tracking-value`.

- Serializer asks the schema registry to give schema details for the combination of topic and key. Schema registry finds this in its cache and sends it.
- The serializer then converts the message to binary format and also stores a schema ID along with the message in binary format.
- When consumer processes the message, it takes the schema ID present in the message.
- The deserializer asks the schema registry to give the schema corresponding to the ID present in the message. Schema registry returns the schema and deserialization can take place.

##### How does the schema end up in registry?

In a non-production environment, the first application that interacts with the new topic can register the schema, but in a production environment, an admin will have to upload them. Both key and value schema for a topic is uploaded into the registry. These are stored in memory, so if something goes wrong, they are lost. To solve this, the schema registry transfers the schema in a special topic in the kafka cluster. In case schema registry crashes, it can create a new instance and connect to the same kafka cluster and the inbuilt consumer can retrieve all the schemas stored in kafka.

Confluent schema registry can be found on github: https://github.com/confluentinc/schema-registry

#### Using Schema registry:

- clone the github repo of the schema registry.
- checkout the latest stable version from the github repo.
- compile: `mvn package`
- schema registry start script can be used to start it: `bin/schema-registry-start config/schema-registry.properties`
- schema registry starts listening for connections on localhost, port 8081.
- Changes to start using schema registry:

After adding the avro serializer and avro dependencies, following changes would have to be done to the producer and consumer

Producer:  
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092, localhost:9093");
props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://localhost:8081");

KafkaProducer<User, Product> myProducer = new KafkaProducer(props);
```
Consumer:  
Similar changes to be done on consumer side also, where the deserializer would need to be changed to `"io.confluent.kafka.serializers.KafkaAvroDeserializer"` and also, `"schema.registry.url"` would need to be added. Another property to be added is `"specific.avro.reader", true` (to cast the received record to appropriate type. We dont need to explicitly register the schema, because that will be done by clients.

## 5. Building Streaming Applications

From a data science perspective, streaming is processing data events one by one as they arrice in our system.

Use cases of Streaming:

- Videos
- Actions/Process execution: Respond to events happening in real time using streaming. For ex: If a customer has entered payment details, and the system needs to process the transaction.
- Data Analytics: Implementing streaming to analyze the traffic in real time is an example.
- Sensor Detection: For example, in case of a fire, we want the system to immediately start sprinklers and call authorities.
- Internet of Things: Interconnecting multiple devices/sensors together and using stream processing and even adding machine learning to enhance capabilities.
- Alerting: A malicious user has accessed your account, you would need to be immediately alerted in order for you to change password.

### Fraud Detection System

Examples: Reject all messages with invalid userId, Only allow items lower than 1,000, total amount greater than 10,000$.  
In a traditional system, these would be steps:
- Payment service responsible to validating and processing the payments
- Payment request comes and the payment service sends a request to a frad detection system to check if the transaction is not fradulent.
- To save all transactions done by the user, we would need to save all these in a DB.
- After all business rules are applied and the validation is successful, transaction is marked as not fradulent.
- If any one of the business rules are not satisfied, the fraud detection service will mark validation unsuccessful, and transaction is marked as a failure. Payment is rejected.

First Bottleneck: There is a dependency between payment service and fraud detection service. One cannot run without the other.  
Second Bottleneck: DB. Since there are 1000s of transactions being processed. Running so many DB calls may result into service becoming unavailable. 

#### How can Kafka help?

- The payment service which is capable of creating and transmitting transactions is connected to a kafka cluster.
- On the other side of the cluster, we have a payment processor which is in charge of all the transactions that arrive in that system.
- Payment service is the Producer producing to the topic: payments, and Payment processor is the consumer reading messages from topic: validated-payments.
- Now we need a fraud detection service, which will identify the valid transactions and pass it onto the validated-payments topic from the payments topic. For this we need to plug in a consumer and a producer.
- There needs to be a business rule set up in the service, so that each transaction passes through this rule.
![Screenshot 2021-06-11 at 7 44 28 PM](https://user-images.githubusercontent.com/10058009/121700374-768dda80-caed-11eb-9d2f-b3d836df18fc.png)

A kafka stream always connects to a kafka cluster. It will not receive messages from any other places. Very common approach is listening from topic A and writing to topic B. The goal of kafka streams is to save us from all the trouble of creating producers and consumers and abstract it all the way in a compact format, which is easy to understand. During the stream processing, the event will undergo a series of operations (topology: chain of operations). Exact definition: Acyclic graph of sources, processors and sinks. The nodes of the graph are called processor, the edge represents a line between the processors, allowing them to go from one processor to another when the previous one has finished. It is acyclic because same message need to processed again.

Different types of processors and steps followed in this specific scenario:  
- Consumer represents a special type of processor called **Source**, which specifies where the stream will extract the data from in order to process it.
- Producer that is present on the end of the processors is called a **Sink**, which sends all the data to the specified location. It is mandatory to have atleast 1 source processor, but the number of sink and stream processors may vary.
- Stream processor: The processors lying between the source and sink. Each stream processor does a specific task, and it can be chained to achieve the desired result. Each stream processor can either be a filter (where specific messages can be filtered), a map (where message is transformed from 1 form to another), a counter (to count all messages of a specific type), of any of several others, which are defined below. Since the count processor needs to store the count of each type of message, we need a "State Store", which can be either ephemeral (if app is down, data is lost), or fault tolerant by persisting the data in external storage. Default is a fault tolerant one, by using an internal topic on the cluster as a storage area. The number of messages of a type can be obtained from the state store and then this count is sent to the sink store as a message.
![Screenshot 2021-06-11 at 8 02 15 PM](https://user-images.githubusercontent.com/10058009/121702954-f321b880-caef-11eb-8ee4-874c435f6510.png)

#### Duality of Streams

In an event-driven architecture, usange of streams might not be enough, we would need to store the data as well. When we process events, we can process them with 2 different perspectives: 1. As a Stream: processing independent event, with no relation to other events (like a user placing multiple orders). **_Delete topics_** can be used as a cleanup policy. 2. As a Database table: Where we persist only the latest state for some specific information. (for example: bank balance is the result of sum of events, always relying upon the previous event). These types of events are stored in _**Compaction topics**_ on Kafka.

To transform a stream to a table, we can perform operations like aggregating, reducing or counting data. To achieve the reverse, we would need to iterate over all the events from beginning of time and store them as independent events.

### Stream Processors

There are 2 categories of processors used in Kafka Streams: Stateless and Stateful. Stateless processor processes each event independently, whereas stateful processor requires a State Store to process events.

Kafka Streams offer a large number of operations that can be performed in a stateless manner. Operations:
- Branch: To split stream in multiple branches based on some business logic.
- Filter: To reject messages based on a condition. 
- Inverse Filter: Opposite of a filter. 
- Map: To transform messages from 1 type to another.
- FlatMap: To transform 1 event to multiple events of same type or different types.
- Foreach: To iterate over each event: This is a terminal process. If foreach is used, we cannot use a sink processor anymore.
- Peek: To inspect elements passing down the stream.
- GroupBy: To group events based on some elements, like key of the message, or an attribute from the value. 
- Merge: To combine 2 streams into a single one.

Complete list: https://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#stateless-transformations

Stateful Operations:
- Aggregations: Example - calulating the sum of all the transactions that have been posted in a topic.
- Count: count messages with the same key.
- Joins: joining streams/tables can be useful when we would like to enhance some messages with information from different topics.
- Windowing: works with intervals of time, on which we can perform various operations.
- Custom Processors: we can create our own processors by using a low-level API.

Complete Details: https://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#stateful-transformations

## 6. Streaming Application with KSQL

Kafka SQL is a SQL streaming engine designed for ease of use by wrapping the Kafka Streams API in a friendly syntax. It has been created by Confluent.

### Why is it needed, when we already have an option?

Easy to create queries without having to worry about the nitty-gritty of Kafka APIs.

Sample KQL Query:  
```sql
CREATE STREAM pineapple_pizza AS
  SELECT crust, size, toppings
  FROM pizza
  WHERE type = 'pineapple';
```

### Developer API's pyramid
![Screenshot 2021-06-12 at 3 43 31 PM](https://user-images.githubusercontent.com/10058009/121772736-fc189580-cb94-11eb-80d1-dda6ff750419.png)

### How does KQL work?

Streaming applications are built inside the KSQL server, which connects to the Kafka cluster and consumes and produces messages by it based on the expressions given by the user. In a non-production environment, the user interacts with a KSQL CLI. The CLI doesn't necessarily have to be on the same machine, because the CLI and KSQL server use REST APIs to communicate. The instructions written on the CLI are sent as REST APIs to the KSQL sever, where statement will be parsed and streaming engine will run it. Each query represents a different streaming application. Every query on the KQL server will be parsed to a topology and then run.

![Screenshot 2021-06-12 at 3 49 13 PM](https://user-images.githubusercontent.com/10058009/121772848-c1fbc380-cb95-11eb-8631-008c33fcd181.png)

```sql
CREATE STREAM pineapple_pizza AS      > .to("pineapple_pizza")
  SELECT crust, size, toppings        > .mapValues( pizza -> pizza.getCrust() + "," + pizza.getSize() + "," + pizza.getToppings())
  FROM pizza                          > .stream("pizza")
  WHERE type = 'pineapple';           > .filter(pizza -> pizza.getType().equals("pineapple"))
```

Valid topology:
```java
.stream("pizza")
.filter(pizza -> pizza.getType().equals("pineapple"))
.mapValues( pizza -> pizza.getCrust() + "," + pizza.getSize() + "," + pizza.getToppings())
.to("pineapple_pizza")
```

### When to use?

All use cases of Kafka Streams are valid here:
- Streaming Operations: Data analytics, Monitoring, IOT, etc.
- Viewing Data: Show content of a topic. With Kafka streams we need to go through a lot of hassle to do the same.
- Manipulating Data: Enhance information stored in a topic. Combining fields, changing them or eliminating them is all just a query away.

### Stream Processing to work with Time Windows

Example Queries: What is the average number of users visiting our website per hour? Total users who order a product per day?

Types of Windowing operations:
- Tumbling
- Hopping
![Screenshot 2021-06-12 at 4 03 04 PM](https://user-images.githubusercontent.com/10058009/121773179-bd380f00-cb97-11eb-9e7b-8deeaeaf51ae.png)

### KSQL Statements

Just like any SQL, KSQL also has DDL and DML statements. 
- Using DDL statements: `CREATE STREAM` and `CREATE TABLE`, we can create either a stream or a table from an underlying Kafka topic. KSQL updates its internal metadata store without making any changes to any Kafka topic. `DROP STREAM` and `DROP TABLE` can be used to delete streams and tables. Other DDL queries are `CREATE STREAM AS SELECT` and `CREATE TABLE AS SELECT`.
- `SELECT`, `INSERT` are common DML statements. `CREATE STREAM AS SELECT` and `CREATE TABLE AS SELECT` are also part of DML category, since these statements define and manipulate data in one go. Data definition part is represented by `CREATE STREAM` and data manipulation part is represente by `AS SELECT`statement.

### Example

Raise an alert is no. of transactions by a user in a 10 min window is more than 5. Below section, explains the solution.

#### Using KSQL:

- Clone the ksql repo: https://github.com/confluentinc/ksql.git
- checkout the latest stable release.
- Compile and package the code: `mvn package`
- update `bootstrap.servers` property inside the config/ksql-server.properties to listen to the right port
- start the ksql server: `bin/ksql-server-start config/ksql-server.properties`
- server should start on port 8088.
- run KSQL CLI: `bin/ksql`. By default the CLI tries to connect to the KSQL server on port 8088 on the same machine, which means no extra arguments need to be passed.
  - run the command `SHOW STREAMS` to see the underlying streams on the KSQL and also the associated kafka topic. Similarly, you can run `SHOW TOPICS` to list all kafka topics on the cluster. If a topic is registered, it will have an entry in the SHOW STREAMS response.
  - To register a topic as a Data Stream, we need to use `CREATE STREAM` query.
    - `CREATE STREAM ksql_payments WITH (KAFKA_TOPIC='payments', VALUE_FORMAT='AVRO');` Before registering the topic we need to make sure that the schema has been registered for this topic.
    - `CREATE TABLE warnings AS SELECT userId, COUNT(*) FROM ksql_payments WINDOW HOPPING (SIZE 10 MINUNTES, ADVANCE BY 1 MINUTE) GROUP BY userId HAVING COUNT(*) > 5;

