# Command
- ### Create topic
    ```
    kafka-topics.sh --create --bootstrap-server <hostname>:9092 --replication-factor 3 --partitions 3 --topic <topic>
    ```
- ### Delete topic
    ```
    kafka-topics.sh --delete --bootstrap-server <hostname>:9092 --topic <topic>
    ```
- ### List topic
    ```
    kafka-topics.sh --list --zookeeper <hostname>:2181
    ```
- ### Check replication and partitions topic
    ```
    kafka-topics.sh --zookeeper <hostname>:2181 --topic <topic> --describe
    ```
- ### Producer
    ```
    kafka-console-producer.sh --broker-list <hostname>:9092 --topic <topic>
    ```
- ### Consumer
    ```
    kafka-console-consumer.sh --bootstrap-server <hostname>:9092  --topic <topic> --from-beginning

    kafka-console-consumer.sh --bootstrap-server <hostname>:9092  --topic <topic> 
    ```
- ### List consumer groups
    ```
    kafka-consumer-groups.sh  --list --bootstrap-server <hostname>:9092
    ```
- ### Check consumer groups
    ```
    kafka-consumer-groups.sh --describe --group <mygroup> --bootstrap-server <hostname>:9092
    ```

- ### Consume data per partition
    ```
    kafka-console-consumer.sh --bootstrap-server <hostname>:9092 --topic <topic> --partition 0 --from-beginning
    ```
- ### Purge a topic

    ```
    kafka-topics.sh --zookeeper <hostname>:2181 --alter --topic <topic> --config retention.ms=100
    ```
    #### รอประมาณ 1-2 นาที 
    ```
    kafka-topics.sh --zookeeper <hostname>:2181 --alter --topic <topic> --delete-config retention.ms
    ```

    - all topic use = ".*"
