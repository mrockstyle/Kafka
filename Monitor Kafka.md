# Monitor Kafka
- ### Download jolokia file
```
wget -O jolokia-agent.jar https://search.maven.org/remotecontent?filepath=org/jolokia/jolokia-jvm/1.6.2/jolokia-jvm-1.6.2-agent.jar
mv  jolokia-agent.jar /data/kafkadata/kafka-bin/libs

```
- ### Configure Kafka 
```
vi /data/kafkadata/kafka-bin/bin/kafka-server-start.sh

```
Put below esac:

```
export JMX_PORT=9999
export RMI_HOSTNAME=<KAFKA_SERVER_IP_ADDRESS>
export KAFKA_JMX_OPTS="-javaagent:/data/kafkadata/kafka-bin/libs/jolokia-agent.jar=port=8778,host=$RMI_HOSTNAME -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=$RMI_HOSTNAME -Dcom.sun.management.jmxremote.rmi.port=$JMX_PORT"

```
- ### Restart Kafka service

```
/data/kafkadata/kafka-bin/bin/kafka-server-stop.sh
/data/kafkadata/kafka-bin/bin/kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties

```
- ### Check jolokia

```
curl http://<ip>:8778/jolokia/version

```
- ### Config telegraf

```
 vi  /etc/telegraf/telegraf.d/jolokia-kafka.conf
 
```
default inputs
```
## Read JMX metrics through Jolokia
 [[inputs.jolokia2_agent]]
   ## An array of Kafka servers URI to gather stats.
   urls = ["http:// <host name>:8778/jolokia"]
   name_prefix = "kafka."

   ## List of metrics collected on above servers
   ## Each metric consists of a name, a jmx path and
   ## optionally, a list of fields to collect.
   ## This collects all heap memory usage metrics.
   [[inputs.jolokia2_agent.metric]]
     name = "heap_memory_usage"
     mbean  = "java.lang:type=Memory"
     paths = ["HeapMemoryUsage"]

   # Kafka Server Replica Fetcher Manager Metrics
   [[inputs.jolokia2_agent.metric]]
     name = "server_replicafetchmanager.maxlag"
     mbean  = "kafka.server:type=ReplicaFetcherManager,name=MaxLag,clientId=Replica"

   [[inputs.jolokia2_agent.metric]]
     name = "BrokerTopicMetrics"
     mbean  = "kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec"   
   [[inputs.jolokia2_agent.metric]]
     name = "TotalTimeMs-Produce"
     mbean  = "kafka.network:name=TotalTimeMs,request=Produce,type=RequestMetrics"
   [[inputs.jolokia2_agent.metric]]
     name = "TotalTimeMs-FetchConsumer"
     mbean  = "kafka.network:name=TotalTimeMs,request=FetchConsumer,type=RequestMetrics"
   [[inputs.jolokia2_agent.metric]]
     name = "TotalTimeMs-FetchFollower"
     mbean  = "kafka.network:name=TotalTimeMs,request=FetchFollower,type=RequestMetrics"
     
   # Kafka Controller Metrics
   [[inputs.jolokia2_agent.metric]]
     name = "controller_activecontrollers"
     mbean  = "kafka.controller:type=KafkaController,name=ActiveControllerCount"



```
topic input (replace your topic in topic name)
```

   [[inputs.jolokia2_agent.metric]]
     name = "BrokerTopicMetrics-<topic name>"
     mbean  = "kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec,topic=<topic name>"
 

```
 