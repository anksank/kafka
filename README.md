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
