# Implement Logstash between Elasticsearch and Filebeat


To achieve our goal of filtering systemd-related logs from the /var/log/dmesg file and sending them to Elasticsearch, we will implement the following steps:

1. Filebeat will be configured to retrieve the complete /var/log/dmesg file.
2. Logstash will then apply a Grok pattern to filter out the systemd-related logs from the file.
3. The filtered logs will be pushed to Elasticsearch for further analysis and storage.


Here's the configuration of the filebat and logstash:

```
#cat /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/dmesg

output.logstash:
  hosts: ["localhost:5044"]
```


```
# cat /etc/logstash/logstash-beat-dmesg.yml
input {
  beats {
    port => 5044
  }
}

filter {
  # For example, to extract systemd-related logs, use the provided Grok pattern
  grok {
    match => { "message" => "\[%{NUMBER:timestamp}\] %{WORD:system}[^\]]+: %{WORD:action} %{GREEDYDATA:message}" }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```

Once it is configure, we have to start the filebeat and logstash with below command:

```
root@osboxes:/usr/share/filebeat/bin# ./filebeat -e -c /etc/filebeat/filebeat.yml

{"log.level":"info","@timestamp":"2023-06-07T05:56:37.826-0400","log.origin":{"file.name":"instance/beat.go","file.line":724},"message":"Home path: [/usr/share/filebeat/bin] Config path: [/usr/share/filebeat/bin]
Data path: [/usr/share/filebeat/bin/data] Logs path: [/usr/share/filebeat/bin/logs]","service.name":"filebeat","ecs.version":"1.6.0"}

root@osboxes:~# /usr/share/logstash/bin/logstash -f /etc/logstash/logstash-beat-dmesg.yml
Using bundled JDK: /usr/share/logstash/jdk
```

Check if the Logstash-DATE index has been created with following command:-

```
osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/_cat/indices?v | grep Logstash
yellow open   Logstash-2023-06-07                  CC89E3D7SbKrz1deXF1Irg   1   1        667            0    410.1kb        410.1kb
osboxes@osboxes:~$
```

Manually pushed the data to the dmesg file and could see that is being fetch by beats and filter by logstash. I was able to view the records after adding a new data view for the index.

```
root@osboxes:/home/osboxes# echo "[    5.455333] systemd[1]: Starting Load/Save Random Seed...[Akshay]" >> /var/log/dmesg
root@osboxes:/home/osboxes#
```

To efficiently discover logs related to systemd in Kibana, follow these steps:

1. Login to Kibana.
2. Navigate to the Kibana UI.
3. Go to the "Discover" section.
4. Select the appropriate index, in this case, the "logstash" index where the logs are stored.
5. Once the index is selected, you will see a list of log entries in the Discover view.

By following these steps, you can effectively explore systemd related dmessage.


![Alt text](https://github.com/AkshaySJadhav/Elastic_Experiments/blob/main/resources/kibana.png)
