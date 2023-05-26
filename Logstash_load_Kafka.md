## Loading Data from Kafka brokers to Elasticsearch with Logstash.


### 1. Download the Kafka.
You can download the kafka from https://archive.apache.org/dist/kafka/ and start the zookeeper and kafka serivice.
```
#wget https://archive.apache.org/dist/kafka/2.7.0/kafka_2.12-2.7.0.tgz
#tar xvzf kafka_2.12-2.7.0.tgz

root@osboxes:~/kafka_2.12-2.7.0/bin# sh zookeeper-server-start.sh -daemon ../config/zookeeper.properties
root@osboxes:~/kafka_2.12-2.7.0/bin# sh kafka-server-start.sh -daemon ../config/server.properties
```

### 2. Create the topic in Kafka and push some data.

We have created a "apache" topice and going to put some test data into the topic from the https://gist.github.com/tdreyno/4278655.
```
root@osboxes:~ /kafka-topics.sh  --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic test

root@osboxes:~/kafka_2.12-2.7.0/bin# sh kafka-topics.sh --describe --zookeeper localhost:2181
Topic: apache	PartitionCount: 1	ReplicationFactor: 1	Configs:
	Topic: apache	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
  
root@osboxes:~/kafka_2.12-2.7.0/bin# sh kafka-console-producer.sh --broker-list localhost:9092 --topic apache < /root/airports.json
>>
```

### 3. Configure the logstash to pull the data from topic.

We are going to create the index "kafka-fligh-data" in this tutorial.

```
osboxes@osboxes:~$ cat /etc/logstash/conf.d/kafka.conf
input {
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => ["apache"]
    }
}
filter {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
}
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "kafka-flight-data"
  }
}

```

### 4. Start the logstash process.
You will start observing database verbose on the screens in sometime that confirms connectivity is working.
```
root@osboxes:/usr/share/logstash/bin# ./logstash -f /etc/logstash/conf.d/mysql.conf
Using bundled JDK: /usr/share/logstash/jdk
[INFO ] 2023-05-26 04:16:15.669 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600, :ssl_enabled=>false}
[INFO ] 2023-05-26 04:16:27.201 [[main]-pipeline-manager] javapipeline - Pipeline Java execution initialization time {"seconds"=>1.59}
[INFO ] 2023-05-26 04:16:27.208 [[main]-pipeline-manager] javapipeline - Pipeline started {"pipeline.id"=>"main"}
[INFO ] 2023-05-26 04:16:29.766 [kafka-input-worker-logstash-0] ConsumerCoordinator - [Consumer clientId=logstash-0, groupId=logstash] Found no committed offset for partition apache-0
```

### 5. Check the Indices.
Once Logstash has finished processing the data, you can verify its completion by checking the indices "kafka-flight-data".


```
root@osboxes:~/kafka_2.12-2.7.0/bin# curl -XGET 127.0.0.1:9200/_cat/indices?v
health status index                                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size

yellow open   .ds-logs-generic-default-2023.05.25-000001 smLN5smCQFylV-HO3yFVXQ   1   1     102972            0     24.3mb         24.3mb
yellow open   shakespeare                                K5ZC2rvnQg6avpilxVysTg   1   1     111396            0     17.6mb         17.6mb
yellow open   microservice-logs                          WM2ZOxbXTpuNXkwpeoP9FA   1   1          3            0     12.2kb         12.2kb
yellow open   kafka-flight-data                          DgfFTNqJToKhGgeoYz_uNQ   1   1      13798            0      1.4mb          1.4mb
root@osboxes:~/kafka_2.12-2.7.0/bin#
````

