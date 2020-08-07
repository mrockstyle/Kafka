# Secure a kafka broker on Datalake Prod.
## User: kafka
- ### Export KAFKA_ZOOKEEPER_CONNECT
```
echo 'export KAFKA_ZOOKEEPER_CONNECT="gsb-lake-prd-mq01:2181"' >> $HOME/.bashrc
source .bashrc
```
- ### Add a super user
```
kafka-configs.sh --zookeeper $KAFKA_ZOOKEEPER_CONNECT --alter --add-config 'SCRAM-SHA-512=[password='admin-secret']' --entity-type users --entity-name admin
```
- ### Create a new file named kafka_server_jaas.conf
```
vi /data/kafkadata/kafka-bin/config/kafka_server_jaas.conf 

KafkaServer {
       org.apache.kafka.common.security.scram.ScramLoginModule required
       username="admin"
       password="admin-secret";
 };
 
KafkaClient {
      org.apache.kafka.common.security.scram.ScramLoginModule required
      username="admin"
      password="admin-secret";
 };
 
Client {
};
```
- ###  Export  kafka_server_jaas.conf
```
echo 'export KAFKA_OPTS="-Djava.security.auth.login.config=/data/kafkadata/kafka-bin/config/kafka_server_jaas.conf -Dzookeeper.sasl.client=false"' >> $HOME/.bashrc
source .bashrc
```
- ### Update the server.properties 
```
vi /data/kafkadata/kafka-bin/config/server.properties

listeners=SASL_PLAINTEXT://gsb-lake-prd-mq01:9092
advertised.listeners=SASL_PLAINTEXT://gsb-lake-prd-mq01:9092
                                                
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.enabled.mechanisms=SCRAM-SHA-512
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-512
 
authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
super.users=User:admin
zookeeper.set.acl=false
```
- ### Restart the kafka server
```
/data/kafkadata/kafka-bin/bin/kafka-server-stop.sh
sleep 3
/data/kafkadata/kafka-bin/bin/kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties
```
- ### Verify secure kafka
```
kafka-console-producer.sh --bootstrap-server gsb-lake-prd-mq01:9092  --topic mymo-eslip-transaction

vi /data/kafkadata/kafka-bin/config/kafka-console-producer.sh

sasl.mechanism=SCRAM-SHA-512
security.protocol=SASL_PLAINTEXT

kafka-console-producer.sh --broker-list gsb-lake-prd-mq01:9092 --topic mymo-eslip-transaction --producer.config=/data/kafkadata/kafka-bin/config/producer.properties
```
#### Note :gsb-lake-prd-mq01, gsb-lake-prd-mq02, gsb-lake-prd-mq03
