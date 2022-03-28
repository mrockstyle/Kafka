# Setup Kafka_2.12-2.5.0 Cluster by shell script
## 1. Download binary Kafka file 
```
wget -O /data/kafka_2.12-2.5.0.tgz https://www-eu.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz
```
### *** If you disconnect to internet. You can scp flie to path /data

## 2. Install Java
- ubuntu 
```
apt-get install openjdk-11-jdk
```     
- redhat
```
yum install java-11-openjdk
yum -y install java-11-openjdk-devel
```    
## 3. Create script setup
- ### ubuntu[user root]
```
vi setup_kafka_cluster_ubuntu.sh
```
```
#!/bin/bash
read -p  "Enter ip broker 0: " broker0
read -p  "Enter hostname broker 0: " hostname0
read -p  "Enter ip broker 1: " broker1
read -p  "Enter hostname broker 1: " hostname1
read -p  "Enter ip broker 2: " broker2
read -p  "Enter hostname broker 2: " hostname2
read -p  "Enter number broker: " numberbroker
read -p  "Enter this hostname : " myhostname
###Config hosts
echo "${broker0} ${hostname0}" >> /etc/hosts
echo "${broker1} ${hostname1}" >> /etc/hosts
echo "${broker2} ${hostname2}" >> /etc/hosts
###Create group and user
addgroup kafka
useradd kafka -u 3041 -s /bin/bash -m -d /home/kafka -g kafka
echo -e "kafka\nkafka" | passwd kafka
chage -I -1 -m 0 -M 99999 -E -1 kafka

###Create working path
tar -xf /data/kafka_2.12-2.5.0.tgz
mkdir -p /data/kafkadata/kafka-bin
mkdir /data/kafkadata/zookeeper
mkdir /data/kafkadata/kafka-logs
cd /data/kafka_2.12-2.5.0
mv *  /data/kafkadata/kafka-bin
rm -rf /data/kafka_2.12-2.5.0
###Update config
sed -i "s/dataDir=\/tmp\/zookeeper/dataDir=\/data\/kafkadata\/zookeeper/g" /data/kafkadata/kafka-bin/config/zookeeper.properties
sed -i "s/admin.enableServer=false/admin.enableServer=true/g" /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "initLimit=5" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "syncLimit=2" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "server.0=${hostname0}:2888:3888" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "server.1=${hostname1}:2888:3888" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "server.2=${hostname2}:2888:3888" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
sed -i "s/broker.id=0/broker.id=${numberbroker}/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/#listeners=PLAINTEXT:\/\/:9092/listeners=PLAINTEXT:\/\/${myhostname}:9092/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/log.dirs=\/tmp\/kafka-logs/log.dirs=\/data\/kafkadata\/kafka-logs/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/num.partitions=1/num.partitions=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/offsets.topic.replication.factor=1/offsets.topic.replication.factor=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/transaction.state.log.replication.factor=1/transaction.state.log.replication.factor=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/transaction.state.log.min.isr=1/transaction.state.log.min.isr=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/zookeeper.connect=localhost:2181/zookeeper.connect=${hostname0}:2181,${hostname1}:2181,${hostname2}:2181/g" /data/kafkadata/kafka-bin/config/server.properties
echo "max.incremental.fetch.session.cache.slots=10000" >> /data/kafkadata/kafka-bin/config/server.properties
###Create myid
echo "${numberbroker}" > /data/kafkadata/zookeeper/myid
chown -R kafka:kafka /data/kafkadata
###Export Kafka path
echo "export KAFKA_HOME=/data/kafkadata/kafka-bin" >> /home/kafka/.bashrc
echo "export PATH=\$PATH:\$KAFKA_HOME/bin" >> /home/kafka/.bashrc
###Create script start kafka
echo "#!/bin/bash" > /home/kafka/start-kafka-server.sh
echo "/data/kafkadata/kafka-bin/bin/zookeeper-server-start.sh -daemon /data/kafkadata/kafka-bin/config/zookeeper.properties" >> /home/kafka/start-kafka-server.sh
echo "sleep 10" >> /home/kafka/start-kafka-server.sh
echo "/data/kafkadata/kafka-bin/bin/kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties" >> /home/kafka/start-kafka-server.sh
chmod 755 /home/kafka/start-kafka-server.sh
chown kafka:kafka /home/kafka/start-kafka-server.sh

echo "Done"
```
```
chmod +x setup_kafka_cluster_ubuntu.sh
```
```
./setup_kafka_cluster_ubuntu.sh
```
```
 vi /etc/rc.local
```
```
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
```
chmod 755 /etc/rc.local
sudo systemctl enable rc-local 
sudo systemctl start rc-local
sudo systemctl status rc-local
```
- ### redhat[user root]
```
vi /data/setup_kafka_cluster_redhat.sh
```
```
#!/bin/bash
read -p  "Enter ip broker 0: " broker0
read -p  "Enter hostname broker 0: " hostname0
read -p  "Enter ip broker 1: " broker1
read -p  "Enter hostname broker 1: " hostname1
read -p  "Enter ip broker 2: " broker2
read -p  "Enter hostname broker 2: " hostname2
read -p  "Enter number broker: " numberbroker
read -p  "Enter this hostname : " myhostname
###Config hosts
echo "${broker0} ${hostname0}" >> /etc/hosts
echo "${broker1} ${hostname1}" >> /etc/hosts
echo "${broker2} ${hostname2}" >> /etc/hosts
###Create group and user
groupadd kafka
useradd kafka -u 3041 -s /bin/bash -m -d /home/kafka -g kafka
echo -e "kafka\nkafka" | passwd kafka
chage -I -1 -m 0 -M 99999 -E -1 kafka
###Create working path
tar -xf /data/kafka_2.12-2.5.0.tgz
mkdir -p /data/kafkadata/kafka-bin
mkdir /data/kafkadata/zookeeper
mkdir /data/kafkadata/kafka-logs
cd /data/kafka_2.12-2.5.0
mv *  /data/kafkadata/kafka-bin
rm -rf /data/kafka_2.12-2.5.0
###Update config
sed -i "s/dataDir=\/tmp\/zookeeper/dataDir=\/data\/kafkadata\/zookeeper/g" /data/kafkadata/kafka-bin/config/zookeeper.properties
sed -i "s/admin.enableServer=false/admin.enableServer=true/g" /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "initLimit=5" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "syncLimit=2" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "server.0=${hostname0}:2888:3888" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "server.1=${hostname1}:2888:3888" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
echo "server.2=${hostname2}:2888:3888" >> /data/kafkadata/kafka-bin/config/zookeeper.properties
sed -i "s/broker.id=0/broker.id=${numberbroker}/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/#listeners=PLAINTEXT:\/\/:9092/listeners=PLAINTEXT:\/\/${myhostname}:9092/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/log.dirs=\/tmp\/kafka-logs/log.dirs=\/data\/kafkadata\/kafka-logs/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/num.partitions=1/num.partitions=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/offsets.topic.replication.factor=1/offsets.topic.replication.factor=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/transaction.state.log.replication.factor=1/transaction.state.log.replication.factor=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/transaction.state.log.min.isr=1/transaction.state.log.min.isr=3/g" /data/kafkadata/kafka-bin/config/server.properties
sed -i "s/zookeeper.connect=localhost:2181/zookeeper.connect=${hostname0}:2181,${hostname1}:2181,${hostname2}:2181/g" /data/kafkadata/kafka-bin/config/server.properties
echo "max.incremental.fetch.session.cache.slots=10000" >> /data/kafkadata/kafka-bin/config/server.properties
###Create myid
echo "${numberbroker}" > /data/kafkadata/zookeeper/myid
chown -R kafka:kafka /data/kafkadata
###Export Kafka path
echo "export KAFKA_HOME=/data/kafkadata/kafka-bin" >> /home/kafka/.bashrc
echo "export PATH=\$PATH:\$KAFKA_HOME/bin" >> /home/kafka/.bashrc
###Create script start kafka
echo "#!/bin/bash" > /home/kafka/start-kafka-server.sh
echo "/data/kafkadata/kafka-bin/bin/zookeeper-server-start.sh -daemon /data/kafkadata/kafka-bin/config/zookeeper.properties" >> /home/kafka/start-kafka-server.sh
echo "sleep 10" >> /home/kafka/start-kafka-server.sh
echo "/data/kafkadata/kafka-bin/bin/kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties" >> /home/kafka/start-kafka-server.sh
chmod 755 /home/kafka/start-kafka-server.sh
chown kafka:kafka /home/kafka/start-kafka-server.sh

echo "Done"
```
```
chmod +x /data/setup_kafka_cluster_redhat.sh
```
```
cd /data/
./setup_kafka_cluster_redhat.sh
```
```
 vi /etc/rc.local
```
```
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
```
chmod +x /etc/rc.d/rc.local
sudo systemctl enable rc-local 
sudo systemctl start rc-local
sudo systemctl status rc-local
```
## 4. Start Kafka when setup all broker
- ### user kafka
```
/home/kafka/start-kafka-server.sh
```
