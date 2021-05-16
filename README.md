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

