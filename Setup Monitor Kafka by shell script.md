# Setup Monitor Kafka by shell script
## 1. Download jolokia file
```bash
wget -O /data/jolokia-agent.jar https://search.maven.org/remotecontent?filepath=org/jolokia/jolokia-jvm/1.6.2/jolokia-jvm-1.6.2-agent.jar
```
### *** If you disconnect to internet. You can scp flie to path /data
## 2. Create script setup
```bash
vi setup_monitor_kafka.sh
```
```bash
#!/bin/bash
read -p  "Enter your hostname : " myhostname
echo "Input topic Ex. \"topic1 topic2 topic3 topic4\""
read -p  "Enter your topic : " mytopic
###mv jar file
mv  /data/jolokia-agent.jar /data/kafkadata/kafka-bin/libs
###Configure Kafka 
sed -i "s/exec \$base_dir\/kafka-run-class.sh \$EXTRA_ARGS kafka.Kafka \"\$@\"//g" /data/kafkadata/kafka-bin/bin/kafka-server-start.sh
echo "export JMX_PORT=9999" >> /data/kafkadata/kafka-bin/bin/kafka-server-start.sh
echo "export RMI_HOSTNAME=${myhostname}" >> /data/kafkadata/kafka-bin/bin/kafka-server-start.sh
echo "export KAFKA_JMX_OPTS=\"-javaagent:/data/kafkadata/kafka-bin/libs/jolokia-agent.jar=port=8778,host=\$RMI_HOSTNAME -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=\$RMI_HOSTNAME -Dcom.sun.management.jmxremote.rmi.port=\$JMX_PORT\"
" >> /data/kafkadata/kafka-bin/bin/kafka-server-start.sh
echo "exec \$base_dir/kafka-run-class.sh \$EXTRA_ARGS kafka.Kafka \"\$@\"" >> /data/kafkadata/kafka-bin/bin/kafka-server-start.sh
### Restart Kafka service
/data/kafkadata/kafka-bin/bin/kafka-server-stop.sh
sleep 5
/data/kafkadata/kafka-bin/bin/kafka-server-start.sh -daemon /data/kafkadata/kafka-bin/config/server.properties
###Check jolokia
curl http://${myhostname}:8778/jolokia/version
###Config telegraf
echo "## Read JMX metrics through Jolokia" > /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo " [[inputs.jolokia2_agent]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   ## An array of Kafka servers URI to gather stats." >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   urls = [\"http://${myhostname}:8778/jolokia\"]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   name_prefix = \"kafka.\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   ## List of metrics collected on above servers" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   ## Each metric consists of a name, a jmx path and" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   ## optionally, a list of fields to collect." >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   ## This collects all heap memory usage metrics." >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   [[inputs.jolokia2_agent.metric]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     name = \"heap_memory_usage\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     mbean  = \"java.lang:type=Memory\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     paths = [\"HeapMemoryUsage\"]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   # Kafka Server Replica Fetcher Manager Metrics" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   [[inputs.jolokia2_agent.metric]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     name = \"server_replicafetchmanager.maxlag\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     mbean  = \"kafka.server:type=ReplicaFetcherManager,name=MaxLag,clientId=Replica\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   [[inputs.jolokia2_agent.metric]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     name = \"BrokerTopicMetrics\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     mbean  = \"kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   [[inputs.jolokia2_agent.metric]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     name = \"TotalTimeMs-Produce\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     mbean  = \"kafka.network:name=TotalTimeMs,request=Produce,type=RequestMetrics\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   [[inputs.jolokia2_agent.metric]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     name = \"TotalTimeMs-FetchConsumer\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     mbean  = \"kafka.network:name=TotalTimeMs,request=FetchConsumer,type=RequestMetrics\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   [[inputs.jolokia2_agent.metric]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     name = \"TotalTimeMs-FetchFollower\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     mbean  = \"kafka.network:name=TotalTimeMs,request=FetchFollower,type=RequestMetrics\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   # Kafka Controller Metrics" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "   [[inputs.jolokia2_agent.metric]]" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     name = \"controller_activecontrollers\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf
echo "     mbean  = \"kafka.controller:type=KafkaController,name=ActiveControllerCount\"" >> /etc/telegraf/telegraf.d/jolokia-kafka.conf

for TOPIC in ${mytopic}
do
   echo "   [[inputs.jolokia2_agent.metric]]">> /etc/telegraf/telegraf.d/jolokia-kafka.conf
   echo "     name = \"BrokerTopicMetrics-${TOPIC}\"">> /etc/telegraf/telegraf.d/jolokia-kafka.conf
   echo "     mbean  = \"kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec,topic=${TOPIC}\"">> /etc/telegraf/telegraf.d/jolokia-kafka.conf
done
```
```bash
chmod +x setup_monitor_kafka.sh
```
## 3. Reload telegraf
```bash
service telegraf reload
```
