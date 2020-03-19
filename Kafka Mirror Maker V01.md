# Kafka Mirror Maker V.1
- ### System Diagram
![GitHub ](https://github.com/mrockstyle/Kafka/blob/master/Kafka%20Mirror%20Maker.JPG)
- ### Installation Kafka
    - Refer to "[Kafka cluster setup](https://github.com/mrockstyle/Kafka/blob/master/Kafka%20cluster.md)"
- ### Configuration in Kafkanode site2 
    - Create /data/kafkadata/kafka-bin/config/site2producer.properties
        ```
        bootstrap.servers=<hostname kafkanode site2>:9092
        max.in.flight.requests.per.connection=1
        batch.size=1000

        ```
    - Create /data/kafkadata/kafka-bin/config/site1consumer.properties
        ```
        bootstrap.servers=<hostname kafkanode site1>:9092
        group.id=<group consumer>
        exclude.internal.topics=true

        ```
- ### Operation
    - Start Kafka Mirror Maker  in Kafkanode site2
        ```
        $ ./kafka-mirror-maker.sh --consumer.config /data/kafkadata/kafka-bin/config/site1consumer.properties --num.streams 2 --producer.config /data/kafkadata/kafka-bin/config/site2producer.properties --whitelist="<topics_mirror>"

        ```

