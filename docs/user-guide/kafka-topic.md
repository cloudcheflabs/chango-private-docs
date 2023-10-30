# Create Kafka Topic

To create kafka topic in Chango Private, first access to the host of kafka nodes.

Login as kafka user and move to kafka installation directory.

```agsl
# login as kafka.
sudo su - kafka;

# move to kafka installation directory.
cd /usr/lib/kafka;
```

Run the command to create topic.

```agsl
export JAVA_HOME=/usr/lib/jdk

bin/kafka-topics \
--create \
--bootstrap-server kafka-host-1:9092,kafka-host-2:9092,kafka-host-3:9092 \
--topic [topic] \
--replication-factor 3 \
--partitions 10 \
--config cleanup.policy=compact
```

`[topic]` is topic name to create.