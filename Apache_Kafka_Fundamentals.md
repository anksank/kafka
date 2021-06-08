# Apache Kafka Fundamentals

*Notes in hardcopy*

## Types of Topics

- **Delete** Topic: Each kafka topic can have a size. If the size of all messages in a topic is equal to the topic size, any new incoming message will result in the oldest message getting deleted. By default there is no size limit set on the topic configuration. Another factor when messages can get deleted is time. Default retention period is 7 days.
- **Compat** Topic: The kafka topic stores message in a Key and Value pair. If there is a message with the same key but different value, the older record with same key is deleted, and newer one is inserted (upsert operation). If the new key does not exist in the topic, its added to the list of messages.

## Creating a Kafka Producer

- Needs 3 Properties: bootstrap.servers, key.serializer, value.serializer
- Can be done using the Properties class from core.java.util library.
  ```java
  Properties props = new Properties();
  props.put("bootstrap.servers", "BROKER-1:9092, BROKER-2:9093");
  props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
  props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
  
  KafkaProducer myProducer = new KafkaProducer(props);
  ```
- Producer connects to the first available broker in the list of brokers supplied in this property, it uses this list to determine the partition owners, or leaders.
  - Best practise to provide more than 1 broker
- Producer is responsible for describing the type of message.
  - String: most common serialization used
- Other producer properties can be found here: http://kafka.apache.org/documentation.html#producerconfigs

## Kafka Producer Records

- Needs 2 values to be set, in order to be a valid `ProducerRecord`: Topic and Value.
- Other optional values are Partition, Timestamp and Key
- Sample: `ProducerRecord myMessage = new ProducerRecord("my_topic", "My Message 1");`
  - 1st parameter is the topic to which message needs to be sent, and 2nd is the message itself. 2nd parameter needs to match the the serializer type for value given in the properties.
  - `ProducerRecord myMessage = new ProducerRecord("my_topic", 3.14);` -> gives a runtime exception
- Sample using other optional parameters:
  ```java
  ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value);
  // Example
  ProducerRecord("my_topic", 1, 124535353325, "Course-001", "My Message 1");
  ```
  - `partition`: can be used when you want to send a message to a specific partition of a topic.
  - `timestamp`: explicit timestamp attached to the message. this can affect performance and throughput in high volume situtations.
    - There are 2 types of timestamps that can be used
      ```java
      // Defined in server.properties:
      log.message.timestamp.type = [CreateTime, LogAppendTime]
      // CreateTime: producer-set timestamp used. Even if not explicitly given by the producer, its attached when message is sent by producer.
      // LogAppendTime: broker set timestamp used when message is appended to the commit log.
      ```
  - `key`: a value to be used as the basis of determining the partitioning strategy to be employed by the kafka Producer. Another purpose of key is that it adds additional information in the message, which can help during the processing. A downside is that additional payload overhead, which will depend on the serialization used.

## Process of Sending Message

- When send method is called, the prodcuer reaches out to the cluster using boostrap.servers list.
- "Metadata" response is returned about the topics, their partitions and managing brokers on the cluster. This instantiates a `Metadata` object inside the producer. This object is always kept fresh with the latest information about the cluster.
- A psuedo processing pipeline starts in the producer. First step is that message is passed through the configured serializer.
- Next step is of the *partitioner*, which decides what partition to use based on the value passed in the `ProducerRecord`.
- Kafka Producer Partitioning Strategy:
  - direct
  - round robin
  - key mod hash
  - custom
![Screenshot 2021-05-16 at 3 21 02 PM](https://user-images.githubusercontent.com/10058009/118393032-6dc6f780-b65a-11eb-9ce7-53494ae29e2a.png)
- Once the partitioning scheme is established, producer dispatches the ProducerRecord onto an in-memory queue like data structure called `RecordAccumulator`: Low level object that has lot of complexity. RecordAccumulator gives the ability to micro-batch(explained below) records.
- When RecordAccumulator receives the ProducerRecord, it gets added to a collection of record batch objects for each topic partition combination needed by the producer instance. Each `RecordBatch` is a small batch of records going to be sent to the broker that owns the assigned partition.
- Properties that are set at the producer level decide the number of records in the RecordBatch.
![Screenshot 2021-05-16 at 4 32 12 PM](https://user-images.githubusercontent.com/10058009/118394775-542aad80-b664-11eb-8de1-0efc6a59a62f.png)
- After message buffering(explained below), when the records are sent to the broker, the broker responds with a `RecordMetadata` Object, which contains information about the records that were successfully or unsuccessfully received.
![Screenshot 2021-05-16 at 5 08 26 PM](https://user-images.githubusercontent.com/10058009/118395694-5b07ef00-b669-11eb-8b76-da378486c252.png)


### Micro-batching in Apache Kafka

Each time you send, persist or read a message, resource overhead is incurred. To make sure this does not cause a bad performance, Kafka uses **micro-batching**.
Small, fast batches of messages, while sending (producer), writing (broker), and receiving (consumer). It makes use of the modern OS system functions like Pagecache and Linux sendfile() system call. By batching, the cost overhead of transmissing, flushing to disk, or doing a network fetch is amortized over the entire batch.

### Message Buffering

- Each RecordBatch has a limit on the number of records that can be buffered. Configuration setting `batch.size` decides the limit. Max bytes that can be buffered each RecordBatch.
- Another setting is `buffer.memory`: the threshold memory (no. of bytes) that will be used to buffer all the RecordBatches.
- If high volume of records being buffered reaches the threshold established in buffer.memory, `max.block.ms` settings comes into effect. The number of milliseconds, the send method will be blocked for. This blocking method forces the thread on the producer to send more ProducerRecords onto the buffer. The hope is that, within the provided number of ms, the buffered contents will be trsansmitted and free up more buffer memory for more records to be enqueued.
- When records are sent to RecordBatch, they wait for one of the 2 things to happen:
  - Record accumulation occurs and when total buffer size reaches the per buffer batch size limit, records are sent immediately in a batch.
  - Simultaneously, new records are being sent to other accumulators and record buffers. Another configuration called `linger.ms` is used: no. of ms an unfull buffer should wait before transmitting whatever records are waiting. In case of high frequency scenarios, linger.ms does not come into the picture.

## Delivery Guarantees

To ensure that the delivery happens, some configuration is done at the producer level.
- Broker acknowledgement ("acks"). Possible values:
  - 0: fire and forget (no acknowledgement is sent by the broker, fastest approach; producer has no way of knowing if the message reached the broker)
  - 1: leader acknowledged (only the leader broker needs to confirm if the message was received, none of the replicated brokers of the quorum; good balance of performance & reliability)
  - 2: replication quorum acknowledged (all replicas must confirm if the message was received; highest level of assurance; at the cost of performance)
- In case of error, no. of "retries" that needs to be attempted can be set.
- "retry.backoff.ms" - wait period in ms, before retrying.

## Ordering Guarantees

- Message ordering is only guaranteed in a specific partition
- Messages sent to multiple partitions will not have a global order. Needs to be handled at the consumer, if required.
- Complication with errors
  - If `retry.backoff.ms` is set to a low value, acknowledgement for a message might not have been received, and a retry would be triggered. Before retry is sent, a 2nd message can be sent, and this causes a problem with ordering.
  - To avoid this, `max.in.flight.request.per.connection` can be set to 1. Any given time, only 1 request can be made. (Ouch)

## Advanced topics that can be explored

- Custom Serializers
- Custom Partitioners
- Asynchronous Send
- Compression
- Advanced Settings - for optimal throughput and performance

## Kafka Consumers

- Just like a Producer, a consumer also needs 3 properties to be instantiated: `bootstrap.servers`, `key` & `value` deserializers (instead of serializers in case of a Producer)
  ```java
  Properties props = new Properties();
  props.put("bootstrap.servers", "BROKER-1:9092, BROKER-2:9093");
  props.put("key.serializer", "org.apache.kafka.common.serialization.StringDeserializer");
  props.put("value.serializer", "org.apache.kafka.common.serialization.StringDeserializer");
  
  KafkaConsumer myConsumer = new KafkaConsumer(props);
  ```
- Other consumer properties can be found here: http://kafka.apache.org/documentation.html#consumerconfigs
- **Subscribing to a Topic:**
  `myConsumer.subscribe(Arrays.asList("my-topic"));`
- single consumer can subscribe to any number of topics.
- a regular expression can also be used as a parameter: `myConsumer.subscribe("my-*");`
- calls to the subscribe method are not incremental: if consumer is subscribed to a topic, you cannot call the same method again to subscribe to another topic (along with the existing topic). It will override the earlier call to subscribe.
- Opposite of subscribe is `unsibscribe`. Example: `myConsumer.unsubscribe();`. Another option to unsubscribe is to pass an empty list to the subscribe method.

## Kafka subscribe and assign API

- calling `subscribe()` method means, dynamic/automatic assignment of partitions. Which means asking single consumer to poll from every partition from the topic. If there are many topics, it can have very important implications.
- `assign` method is used to subscribe to individual partitions. This way is manual, self-administering mode. Irrespective of ther topics, an instance of consumer can be assigned to listen to any specific partition. This method also takes input as list and method cannot be called incrementally. `Consumer Groups` take care of this assignment for us.
- Example:
  ```java
  TopicPartition partition0 = new TopicPartition("myTopic", 0); // TopicPartition is a class to represent a partition safely in Java
  ArrayList<TopicPartition> partitions = new ArrayList<TopicPartition>();
  partitions.add(partition0);
  
  myConsumer.assign(partitions);
  ```

## Single Consumer Topic Subscriptions: comparison between subscribe and assign

- Benefit of using the subscribe method to poll messages is that, the partition management is entirely managed for you. (for ex: when new partition is added to a topic, the metadata gets updated and the consumer is notified. Consumer has an internal object called `SubscriptionState` which manages the subscriptions, it knows if a change affects the subscriptions).
- In case of `assign` method call, the consumer does not care if there is a new partition added to an existing topic, although it does get informed about it.

## The Poll Loop

Nothing happens until you start `poll()` loop. Its the primary function of a Kafka Consumer.
- When poll() is called, the consumer starts polling the brokers for data.
- Its the single API for handling all consumer broker interactions.
- This is in fact an infinite loop, which will only be interrupted for valid reasons.
  ```java
  myConsumer.subscribe(topics);
  
  try {
    while (true) {
      ConsumerRecords<String, String> records = muConsumer.poll(100);
      // processing logic goes here ...
    }
  }
  finally {
    myConsumer.close();
  }  
  ```

### Kafka Producer perf test shell program

Sample: Send 50 records, 1 byte each, 10 per second.  
`bin/kafka-producer-perf-test.sh --topic my-other-topic --num-records 50 --record-size 1 throughput 10 producer-props bootstrap.servers=localhost:9092 key.serializer=org.apache.kafka.common.serialization.StringSerializer value.serializer=org.apache.kafka.common.serialization.StringSerializer`

### Kafka topics shell program to alter existing topics

To alter the number of paritions:  
`bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic my-topic --partitions 4`

## Kafka Consumer Polling

- When subscribe/assign is called, `SubscriptionState` object is used to keep the current information about all the topics and partitions a consumer is assigned to. This object also plays an important role along with `ConsumerCoordinator` to manage offsets.
- When poll(100) is invoked, consumer settings related to the bootstrap server is used to request the metadata of the cluster.
- `Fetcher` is the most important object which takes care of the communication between consumer and the cluster. Within this, there are several fetch related operations to initiate communication. 
- Fetcher communicates to the cluster through Consumer Network Client. When the client is open sending TCP packets, consumer starts sending heartbeats, which enables the cluster to know what consumers are connected. Initial metadata is also sent this time. This is used to update the internal metadata object and is updated till the time poll method is running and whenever the cluster details change.
- `ConsumerCoordinator` now takes the responsibility to coordinate between the consumers. It has 2 main jobs:
  - Being aware of automatic/dynamic partition re-assignments and notification of the assignment changes to the SubscriptionState object.
  - Committing offsets to the cluster. Once the offset is confirmed by the cluster, the SubscriptionState object is also notified of the same.
- To start retrieving messages, the fetcher needs to know what topics and paritions it should be asking for. This info comes from the SubscriptionState object and then starts asking for messages.
- The value passed in the poll method (100) -> This is the number of milliseconds for which the network waits to receive messages from the cluster before returning. Its the minimum amount of time each message retrieval cycle will take.
- When timeout expires, a batch of records is returned and added to the in memory buffer where they are parsed, deserialized and grouped into consumer records by topic and partition. Once the fetcher finishes this process, it returns the object for processing.
- **poll() process is single-threaded operation**. There can only be a single thread per kafka consumer.
![Screenshot 2021-06-04 at 1 26 11 PM](https://user-images.githubusercontent.com/10058009/120767073-78c5c700-c538-11eb-98f6-e84fb471448a.png)  
- Never spend too much time processing the result from the consumer. Hence, having a single consumer might not be the best idea.

## Consumer Offsets

- Value that enables consumers to operate independently by representing the last read position that the consumer has read from a partition of a topic.
- Different categories of offsets, each representing a different state:
  - `last committed offset`: the last record that the consumer has confirmed to have processed. For any given topic, a consumer may have multiple offsets its tracking, since each partition is mutually exclusive from the other.
  - `current position`: as the consumer reads messages from the last committed offset, it tracks its current position.
  - `log-end offset`: current position advances as the consumer advances towards the last record in the partition, which is the log-end offset.
  - `un-committed offsets`: difference between the current position and last committed offset.
- We need to reduce the un-committed offsets value by programming in the right way.
![Screenshot 2021-06-04 at 1 37 50 PM](https://user-images.githubusercontent.com/10058009/120768715-1372d580-c53a-11eb-9bec-444b61cf4864.png)
- Property that takes care of updating the last committed offset: `enable.auto.commit`, default value for which is true. Its a blind setting, because kafka does not know under what logical circumstances, a record can be considered processed. It does this updation using another property called `auto.commit.interval`, with default value of 5000 (ms).
  - In case the processing logic is not able to process a ConsumerRecord within 5 seconds, the consumer is going to notify the customer that the record is processed. If for some reason, the processing fails, there would be no way to go back to the **actually** committed offset.

**The extent in which your system can be tolerant of, eventually consistency is determined by its reliability.**

### Offset Behaviour

- Just because something is read, doesn't mean that its committed. **Read != Committed**
- Offset behaviour is configurable.
  - convenient option is the default one provided by kafka: `enable.auto.commit = true`. Its comparable to garbage collection in modern programming languages.
  - In kafka, we can use the commit frequency to take care of this. `auto.commit.internal.ms = 5000`. Increasing this to a upper bound which your processing logic can take, will take care of the problem. But it creates a offset gap in the opposite direction where your commits are lagging behind your processing positions.
  - another property: `auto.offset.reset = "latest"` (default). When a consumer starts reading from a new parition. In contrast, this could be set to `"earliest"`. Another setting is `"none"`, when you want to decide which record you want to process.
- **Storing the offsets**: Kafka stores the committed offsets in a special topic called, `__consumer_offsets`. It has 50 partitions by default. The ConsumerCoordinator takes care of producing messages to be written to this topic.

### Offset Management

- There are 2 modes of offset management, automatic and manual. To choose manual, the property `enable.auto.commit` needs to be set to false. This gives the full control of offset commits to the consumer application. The API to take care of this has 2 properties: `commitSync()` and `commitAsync()`.
- commitSync: used when you want precise control over when you want to inform the cluster that a record has been processed. `myConsumer.commitSync();` It would make sense if this step is done after the processing of your batch of records is done, not after individual processing of records, since the call is a blocking one and will wait till there is a response back from the cluster. commitSync is retried till there is success or till an unrecoverable error. Property used to retry commitSync: `retry.backoff.ms` (default:100) -> time after which you want retry to be triggered. In this case, throughput and performance is traded for control and consistency.
- commitAsync: non-blocking call, but non-deterministic. There are no retries in this case when the commit does not happen. However, there is a callback option to be passed as shown below. Throughput and overall performance is better. 
  example:
  ```java
  try {
    for (...) { // Processing batches of records... }
    // Not recommended:
    myConsumer.commitAsync();
    
    // Recommended:
    myConsumer.commitAsync(new OffsetCommitCallback() {
      public void onComplete(..., ..., ...) { // do something }
    });
  }
  ```

## Committing Offsets

- Offset Management occurs when the poll method has timed out and the records have been presented for processing. Whether its an automatic commit or explicit call to commit API, the commit process takes a batch of records, determine the offsets and asks the ConsumerCoordinator to commit them to the Kafka Cluster via the Consumer Network Client.
- When the offsets have been confirmed to have been committed, ConsumerCoordinator updates the SubscriptionState object accordingly, so the fetcher is always aware of the offset of committed records and what is the next record to be retrieved.
![Screenshot 2021-06-05 at 2 59 46 PM](https://user-images.githubusercontent.com/10058009/120887104-b1d06b00-c60e-11eb-839f-d00bdfe7f000.png)


### Common reasons of taking control of the offset management

- Consistency Control
  - When is "done"
- Atomicity: being able to process the records as an atomic operation. Specially in transaction processing system.
  - This is required when there is a desire to reach "Exactly once" message processing vs. "At-least-once" message processing.

#### Single execution thread is definitely not enough to receive and process the messages retrieved from multiple paritions of a topic.

## Scaling out Customers (Consumer Groups)

- If more message production is needed, solution is to add more producers
- If more message retention and redeundancy is needed, add more brokers
- If more metadata management facilities is needed, add more zookeeper members
- If more processing and scalability is required, `Consumer Groups` are required. (Consumer Side scale-out)
  - Independent consumers working as a team
  - Only thing required to join a consumer group is the `group.id` setting before starting a consumer
  - The task of message consumption and processing load is evenly balanced amongst the consumers.
    - Parallelism and throughput increases
    - Redundancy level can be increased
    - Performance is better

### Steps when multiple consumers form a consumer group

- When multiple consumers with the same group.id calls the subscribe method with the same list of topics, behind the scene, a designated broker is elected as `GroupCoordinator` whose job is to manage and maintain consumer groups membership. GroupCoordinator works with the Cluster Coordinator and zookeeper to assign and monitor specific paritions within a topic to individual consumers within a consumer group.
- After a consumer group is formed, each consumer sends a regular heartbeats at an interval defined in property `heartbeat.internal.ms = 3000`. Based on this heartbeat, the group coordinator decides whether an individual consumer is alive and able to participate in the group.
- GroupCoordinator waits for a specific time called `session.timeout.ms = 30000` for any consumer to send heartbeat, after which the consumer is treated as failed and corrective measures are taken. Main task of GroupCoordinator is to make sure the load is balanced among all the consumers of a consumer group.
- If a consumer is not available for some reason, the GroupCoordinator removes that consumer and assigns the partition to some other consumer of the group. This is called `Consumer Rebalance`. If there is no additional consumers in the consumer group, the first consumer in the group gets the new assignment and in this case ends up taking twice as much load as before.
- Similar rebalance occurs if there is a consumer added, or a partition is added to the cluster.
![Screenshot 2021-06-06 at 5 09 16 PM](https://user-images.githubusercontent.com/10058009/120923027-f3821400-c6e9-11eb-83f5-d0d2c05d0efc.png)

### Consumer Group Rebalancing

When a new consumer joins a group and a parition which was earlier assigned to some other consumer is assigned to the new one, the new consumer needs to know the position from which it needs to poll messages. This is done using the property defined in the `auto.offset.reset` setting (default: latest). Hence, new consumer starts reading from the last committed offset of the previous consumer (assuming that the previous consumer was able to commit this information properly). If the previous consumer was processing a record which it was not able to commit, its possible that the new consumer picks up a record which was already processed. This creates duplicates.

### Importance of Group Coordinator

- Evenly balance available Consumers to partitions.
  - If its possible, it will assign 1 partition to 1 consumer (1:1 Consumer-to-partition ratio)
  - If there are extra consumers compared to number of paritions, they are going to be idle.
- When a new parition becomes available, the GroupCoordinator initiates the rebalancing protocol by engaging each ConsumerCoordinator in the impacted consumers, to start process of rebalancing
  - When partition is added
  - Consumer failure

## Consumer Configuration

Consumer performance and efficiency
- `fetch.min.bytes`: minimum number of bytes that must be returned from the poll, to ensure that there are no wasted cycles of processing if there are not enough messages to process. Analogous to `batch.size` on the producer side.
- `max.fetch.wait.ms`: time to wait if there is not enough data available which is set in the fetch.min.bytes setting. Similar to linger.ms setting on the producer.
- `max.partition.fetch.bytes`: to make sure your processing logic is not overloaded, this property can be set, which is the maximum number of bytes to be fetched per partition.
- `max.poll.records`: Similar to the above setting. maximum no. of records that can be polled in each cycle. The previous 2 settings are useful to throttle the batch of record received by the consumer at once (when processing time is a concern)

## Advanced topics to be explored:
- Consumer position control -> 3 methods: 
  - seek() -> to read a specific offset of message in a given topic and parition.
  - seekToBeginning() -> when you want start reading from the beginning of a group of specific topics and paritions
  - seekToEnd() -> opposite of seekToBeginning()
- Flow Control -> If you want to pause specific topics and parition, and process other topic and paritions (higher priority). Helpful when a single consumer has to read from multiple topics and partitions
  - pause()
  - resume()
- Rebalance listeners: These listeners notify you when a rebalance occurs, in case you want to handle offsets yourself.

## Primary use cases for Apache Kafka

- Connecting disparate sources of data: its possible to write data connectors for practically any data source
- Large scale data movement pipelines: can be a replacement of long standing, expensive, fragile ETL environments
- "Big Data" integration: integration with Hadoop and Spark is very much possible

### Challenges and whats next? (2016)

- Governance and Data Evolution
  - In advanced cases, the built in serializer for Key and Value are not enough. When diversity increases among the systems, custom serializers come to play. The consumers need to understand the data being produced and needs to be aware of the deserialization to be used to make sense out of the data. Challenge here is the cataloging and management of message serializers being used on the producer side and deserailizers needed on the consumer side.
![Screenshot 2021-06-06 at 9 54 55 PM](https://user-images.githubusercontent.com/10058009/120932053-df520d00-c711-11eb-87e0-8527ce819b54.png)
  - One of biggest kafka ecosystem contributor is **Confluent** and they have recognised this challenge. They introduced **Kafka Schema Registry**. How?
    - Apache Avro serialization format: version format (producers can serialize messages in an avro versioned and self describing format and expect them to be deserialized by the consumers)
    - Schema registry and version management: the versioned schema used can be maintained centrally on the cluster used.
    - RESTful service discovery
    - Compatibility broker
    - Open source with Apache license
- Consistency and Productivity: need for lower overhead and less investment. hence, inconsistency and cost remain a challenge in case of Kafka.
  - Lot of duplicate applications are created that connect specific data sources together. The challenge is lack of a consistent framework for providing connectivity between specific data sources and targets. Its always left to the individual to create these integration scenarios, which is more cost and maintenance. Not every company has resources to maintain this.
![Screenshot 2021-06-06 at 10 03 14 PM](https://user-images.githubusercontent.com/10058009/120932353-05c47800-c713-11eb-9d84-adfb8fbcf667.png)
  - 0.10 release addressed this challenge: **Apache Kafka Connect**
    - API for developers which gives common framework for integration: Standardization of common approaches of Producers and Consumers.
    - Oracle and HP are examples
    - 50+ connectors available (thanks to Confluent for major contribution)
    - There is a connector Hub that has all details. It has become cheaper and fast to use the Kafka Connect.
- Big and Fast data: management of big data in a fast way is going to be a big challenge in the coming years.
  - There are multiple solutions to deal with Machine Learning, Predictive Analysis in a real time environment like Apache Storm, Apache Cassandra, Apache Spark, Apache Hadoop. Kafka is found in the middle. Each of these technologies have their own set of APIs to manage cluster and perform other operations. If all these technologies exist under the same roof, there are consistency and productivity challenges in integrating it all together. With kafka in the center, it would need an army of producers and consumers to keep the pipeline flowing.
![Screenshot 2021-06-06 at 11 08 19 PM](https://user-images.githubusercontent.com/10058009/120934505-1d542e80-c71c-11eb-93d2-6754b13b15c4.png)
  - With 0.10 release, a client library for real time stream based processing was introduced: **Kafka Streams**
    - Leverages existing Kafka machinery. The products using all the components for real time data analysis, can now have streaming data capabilities without having to install, run and maintain all of the above mentioned platforms.
    - Single infratructure solution. Although many organisations still invest in apache hadoop and spark for various reasons. At least for stream based processing, Kafka could be the only system needed.
    - Kafka streams is a client library, so its just like kafka producers and consumers, which can be embedded in existing applications.

#### Netflix, LinkedIn, Confluent, Twitter, Uber -> Huge users of Kafka and also make generous contributions to it.
