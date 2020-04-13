## Kafka Standalone Setup
- ### Kafka Version: Kafka_2.12-2.3.0 
	(https://www.apache.org/dyn/closer.cgi?path=/kafka/2.3.0/kafka_2.12-2.3.0.tgz)
- ### Dependency: JDK11 (8 or later)
- ### Minimum spec(per node):
	- Cpu – 2 cores 
 	- Memory - 4 GB 
	- Data disk (/data) – 50 GB
- ### System Diagram
	![GitHub ](https://github.com/mrockstyle/Kafka/blob/master/kafka%20standalone.JPG)

- ### Config hosts [**user root**]
     ```
     $ sudo vi /etc/hosts

        <ip1> <hostname1>

     ```
- ### Create group and user [**user root**]
     - ubuntu 
          ```
          $ addgroup kafka
          $ useradd kafka -u 3041 -s /bin/bash -m -d /home/kafka -g kafka
          $ echo -e "kafka\nkafka" | passwd kafka
          $ chage -I -1 -m 0 -M 99999 -E -1 kafka

          ```     
     - redhat
          ```
          $ groupadd kafka
          $ useradd kafka -u 3041 -s /bin/bash -m -d /home/kafka -g kafka
          $ echo -e "kafka\nkafka" | passwd kafka
          $ chage -I -1 -m 0 -M 99999 -E -1 kafka

          ```    
- ### Install Java [**user root**]
     - ubuntu 
          ```
          $ sudo apt-get install openjdk-11-jdk

          ```     
     - redhat
          ```
          $ yum install java-11-openjdk

          ```    

- ### Download binary Kafka file [**user root**]
     - ubuntu
          ```
          $ cd /data
          $ wget https://www-eu.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz

          ```
     - redhat 
          ```
          $ cd /data
          $ yum install wget
          $ yum install nmap-ncat
          $ wget https://www-eu.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz

          ```
- ### Create working path [**user root**]
     ```
     $ tar -xf kafka_2.12-2.3.0.tgz
     $ mkdir -p /data/kafkadata/kafka-bin
     $ mkdir /data/kafkadata/zookeeper
     $ mkdir /data/kafkadata/kafka-logs
     $ cd /data/kafka_2.12-2.3.0
     $ mv *  /data/kafkadata/kafka-bin
     $ cd /data
     $ rm -rf kafka_2.12-2.3.0
     $ chown -R kafka:kafka kafkadata

     ```
- ### Update config [**user kafka**]
     - [path /data/kafkadata/kafka-bin/config]
          ```
          $ vi zookeeper.properties

               # change zookeeper working path
               dataDir=/data/kafkadata/zookeeper

          ```
          ```
          $ vi server.properties

               #update kafka configuration
               Broker.id=0 
               log.dirs=/data/kafkadata/kafka-logs
	          listeners = PLAINTEXT://<hostname>:9092
               num.partitions=1
               log.retention.hours=168
               offsets.topic.replication.factor=1
               transaction.state.log.replication.factor=1
               transaction.state.log.min.isr=1
               zookeeper.connect=<hostname1>:2181

          ```
- ###  Export Kafka path [**user kafka**]
     ```
     $ vi .bashrc

          export KAFKA_HOME=/data/kafkadata/kafka-bin
          export PATH=$PATH:$KAFKA_HOME/bin

     $ source .bashrc

     ```
- ### Start Zookeeper [**user kafka**]
     ```
     $ zookeeper-server-start.sh -daemon /data/kafkadata/kafka-bin/config/zookeeper.properties

     ```
- ### Start server [**user kafka**]
     ```
     $ kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties

     ```
- ### Check on each node 
     - Check mode
          ```
        $ echo stat |nc <hostname1> 2181|grep Mode

          ```
          Must show standalone mode
