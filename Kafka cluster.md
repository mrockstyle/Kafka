# Kafka Cluster Setup
- ### 
- ### Config hosts
     ```
     $ sudo vi /etc/hosts

    <ip1> <hostname1>
    <ip2> <hostname2>
    <ip3> <hostname3>
     ```
- ### Create group and user Kafka
     ```
     $ addgroup kafka
     $ useradd kafka -u 3041 -s /bin/bash -m -d /home/kafka -g kafka
     $ echo -e "kafka\nkafka" | passwd kafka
     $ chage -I -1 -m 0 -M 99999 -E -1 kafka

     ```     
- ###  Export Kafka path [**user kafka**]
     ```
     $ vi .bashrc

          export KAFKA_HOME=/data/kafkadata/kafka-bin
          export PATH=$PATH:$KAFKA_HOME/bin

     $ source .bashrc

     ```
- ### Download binary Kafka file [**user kafka**]
     ```
    $ cd /data
    $ wget https://www-eu.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz

     ```
- ### Create working path [**user kafka**]
     ```
     $ mkdir -p /data/kafkadata/kafka-bin
     $ mkdir /data/kafkadata/zookeeper
     $ mkdir /data/kafkadata/kafka-logs
     $ cd /data/kafka_2.12-2.3.0
     $ mv *  /data/kafkadata/kafka-bin

     ```
- ### Update config [**user kafka**]
     - [path /data/kafkadata/kafka-bin/config]
          ```
          $ vi zookeeper.properties

               # change zookeeper working path
               dataDir=/data/kafkadata/zookeeper
               # add configuration for zookeeper cluster 
               initLimit=5
               syncLimit=2
               server.0=<hostname1>:2888:3888
               server.1=<hostname2>:2888:3888
               server.2=<hostname3>:2888:3888

          ```
          ```
           $ vi server.properties

               #update kafka configuration
               Broker.id=0 or 1 or 2
               listeners = PLAINTEXT://<hostname>:9092
               log.dirs=/data/kafkadata/kafka-logs
               num.partitions=3
               log.retention.hours=168
               offsets.topic.replication.factor=3
               transaction.state.log.replication.factor=3
               transaction.state.log.min.isr=3
               zookeeper.connect=<hostname1>:2181,<hostname2>:2181,<hostname3>:2181

          ```
- ###  Create myid [**user kafka**]
     ```
     $ cd /data/kafkadata/zookeeper
     $ vi myid

          0 or 1 or 2

     ```
- ### Start Zookeeper [user kafka]
     ```
     $ ./zookeeper-server-start.sh -daemon /data/kafkadata/kafka-bin/config/zookeeper.properties

     ```
- ### Start server [user kafka]
     ```
     $ ./kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties

     ```
- ### Check on each node 
     - Check leader node
          ```
          $ echo stat |nc <hostname1> 2181|grep Mode
	     $ echo stat |nc <hostname2> 2181|grep Mode
		$ echo stat |nc <hostname3> 2181|grep Mode

          ```
     - On zookeeper leader node
          ```
          $ echo mntr | nc <leader node> 2181|grep follower

          ```
          Must have 2 zk_followers and 2 zk_synced_followers
