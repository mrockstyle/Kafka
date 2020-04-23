# Upgrade Kafka_2.12-2.3.0 to Kafka_2.12-2.4.1




- ### ทำทุกขั้นตอนให้ครบทั้ง 3 broker ก่อนทำขั้นตอนถัดไป

- ### Kafka Version: Kafka_2.12-2.4.1 
	(https://www.apache.org/dyn/closer.cgi?path=/kafka/2.4.1/kafka_2.12-2.4.1.tgz)


- ### Update old config and restart zookeeper [**user kafka**]
    #### Update old config
    - [path /data/kafkadata/kafka-bin/config]
        ```
	    $ cd /data/kafkadata/kafka-bin/config
        $ vi zookeeper.properties

            snapshot.trust.empty=true

        ```
    #### Restart zookeeper 
    - [path /data/kafkadata/kafka-bin]
        ```
	    $ cd /data/kafkadata/kafka-bin
        $ bin/zookeeper-server-stop.sh
        $ bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

        ```
- ### Download binary Kafka_2.12-2.4.1 file [**user kafka**]

    ```
    $ cd /data/kafkadata
    $ wget https://downloads.apache.org/kafka/2.4.1/kafka_2.12-2.4.1.tgz

    ```
- ### Create new working path and copy old config to new working path [**user kafka**]
     ```
     $ tar -xf kafka_2.12-2.4.1.tgz
     $ cd /data/kafkadata/kafka-bin/config
     $ cp zookeeper.properties /data/kafkadata/kafka_2.12-2.4.1/config
     $ cp server.properties /data/kafkadata/kafka_2.12-2.4.1/config
     $ cd /data/kafkadata/kafka-bin/libs
     $ cp jolokia-agent.jar data/kafkadata/kafka_2.12-2.4.1/libs
    
     ```
- ### Update new config [**user kafka**]
    - [path /data/kafkadata/kafka_2.12-2.4.1/config]
        ```
	    $ cd /data/kafkadata/kafka_2.12-2.4.1/config
        $ vi server.properties

            inter.broker.protocol.version=2.3.0

        ```
- ### Stop/Start server,zookeeper and change new version [**user kafka**]

    #### Stop server and zookeeper
    - [path /data/kafkadata/kafka-bin]
        ```
	    $ cd /data/kafkadata/kafka-bin
        $ bin/kafka-server-stop.sh
        $ bin/zookeeper-server-stop.sh

        ```
    #### Change new version 
    - [path /data/kafkadata]
        ```
	    $ cd /data/kafkadata
        $ mv kafka-bin kafka-bin-bac
        $ mv kafka_2.12-2.4.1 kafka-bin

        ```
    #### Start server and zookeeper
    - [path /data/kafkadata/kafka-bin]
        ```
	    $ cd /data/kafkadata/kafka-bin
        $ bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
        $ bin/kafka-server-start.sh -daemon config/server.properties

        ```
- ### Update new config for the new protocol version and restart server [**user kafka**]
    #### Update new config
    - [path /data/kafkadata/kafka-bin/config]
        ```
	    $ cd /data/kafkadata/kafka-bin/config
        $ vi server.properties

            inter.broker.protocol.version=2.4.1

        ```
    #### Restart server
    - [path /data/kafkadata/kafka-bin]
        ```
	    $ cd /data/kafkadata/kafka-bin
        $ bin/kafka-server-stop.sh
        $ bin/kafka-server-start.sh -daemon config/server.properties

        ```
