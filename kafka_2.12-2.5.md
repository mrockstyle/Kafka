# Kafka Cluster Setup

- ### Kafka Version: Kafka_2.12-2.5.0 
	(https://www.apache.org/dyn/closer.cgi?path=/kafka/2.5.0/kafka_2.12-2.5.0.tgz)
- ### Dependency: JDK11 (8 or later)
- ### Minimum spec(per node):
	- Cpu – 2 cores 
 	- Memory - 4 GB 
	- Data disk (/data) – 50 GB
- ### System Diagram

	![GitHub ](https://github.com/mrockstyle/Kafka/blob/master/kafka%20cluster.JPG) 

- ### Config hosts [**user root**]
     ```
     $ sudo vi /etc/hosts
     
      <ip1> <hostname1>
      <ip2> <hostname2>
      <ip3> <hostname3>
	
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
          $ yum -y install java-11-openjdk-devel


          ```    
- ### Download binary Kafka file [**user root**]
     - ubuntu
          ```
          $ cd /data
          $ wget https://www-eu.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz

          ```
     - redhat 
          ```
          $ cd /data
          $ yum install wget
          $ yum install nmap-ncat
          $ wget https://www-eu.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz

          ```
- ### Create working path [**user root**]
     ```
     $ tar -xf kafka_2.12-2.5.0.tgz
     $ mkdir -p /data/kafkadata/kafka-bin
     $ mkdir /data/kafkadata/zookeeper
     $ mkdir /data/kafkadata/kafka-logs
     $ cd /data/kafka_2.12-2.5.0
     $ mv *  /data/kafkadata/kafka-bin
     $ cd /data
     $ rm -rf kafka_2.12-2.5.0
     $ chown -R kafka:kafka kafkadata

     ```

- ### Update config [**user kafka**]
     - [path /data/kafkadata/kafka-bin/config]
          ```
          $ vi zookeeper.properties

               # change zookeeper working path
               dataDir=/data/kafkadata/zookeeper
               # add configuration for zookeeper cluster 
               admin.enableServer=true
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
               max.incremental.fetch.session.cache.slots=10000


          ```
- ###  Create myid [**user kafka**]
     ```
     $ cd /data/kafkadata/zookeeper
     $ vi myid

          0 or 1 or 2

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
- ### Create script start kafka [**user kafka**]
     ```
     $ cd /home/kafka
     $ vi /home/kafka/start-kafka-server.sh
        #!/bin/bash
        /data/kafkadata/kafka-bin/bin/zookeeper-server-start.sh -daemon /data/kafkadata/kafka-bin/config/zookeeper.properties
        sleep 10
        /data/kafkadata/kafka-bin/bin/kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties

     $ chmod 755 start-kafka-server.sh
     
     ```
- ### Config rc.local [user root]

     ```
     $ vi /etc/rc.local
        #!/bin/bash
        # THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
        #
        # It is highly advisable to create own systemd services or udev rules
        # to run scripts during boot instead of using this file.
        #
        # In contrast to previous versions due to parallel execution during boot
        # this script will NOT be run after all other services.
        #
        # Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
        # that this script will be executed during boot.

        echo "kafka" | su - kafka -c "/home/kafka/start-kafka-server.sh"

     ```
    - ubuntu
        ```
          $ chmod 755 /etc/rc.local	

        ```
    - redhat 
        ```
         $ chmod +x /etc/rc.d/rc.local
        ```
     ```
     $ sudo systemctl enable rc-local 
     $ sudo systemctl start rc-local
     $ sudo systemctl status rc-local
     
     ```