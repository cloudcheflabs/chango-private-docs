# Configure Schema Registry URL

Schema Registry is schema registry server for kafka to handle schemas of kafka messages. 
For example, if you want to send Avro messages to kafka, then first, you need to register Avro schema to Schema Registry.

To send Avro messages to Kafka using Schema Registry, see <a href="https://itnext.io/howto-produce-avro-messages-to-kafka-ec0b770e1f54">here</a> for more details.

The following property needs to be configured to Kafka producer and consumer.

```agsl
schema.registry.url=http://[schema-registry-host]:8081
```

To find the host of Schema Registry, go to `Components` -> `Apache Kafka`. 
The host name will be found in `Schema Registry Host` of `Status` section.
