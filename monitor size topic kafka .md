# Monitor size topic kafka
- ### Create script [user kafka]
    ```
    vi /home/kafka/disk_exec_du.sh
    ```
    ```
    #!/usr/bin/env bash
    du -s "${1}" | awk '{print "[ { \"Kbytes\": "$1", \"dudir\": \""$2"\" } ]";}'
    ```
    ```
    chmod 755 /home/kafka/disk_exec_du.sh
    ```
- ### Create cofig telegraf [user root]
    ```
    vi /etc/telegraf/telegraf.d/disk_topic_kafka.conf
    ```
    ```
    [[inputs.exec]]
    commands = [ "/home/kafka/disk_exec_du.sh /data/kafkadata/kafka-logs/<topic>]
    timeout = "5s"
    name_override = "kafka.disk-topic"
    name_suffix = ""
    data_format = "json"
    tag_keys = [ "dudir" ]

    ```
- ### Reload config telegraf
    ```
    service telegraf reload
    ```


