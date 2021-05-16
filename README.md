# Apache Kafka

*Notes in hardcopy*

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
  - `key`: a value to be used as the basis of determining the partitioning strategy to be employed by the kafka Producer. Another purpose of key is that it adds additional information in the message, which can help during the processing. A downside is that additional payload is used, this will depend on the serialization used.

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
