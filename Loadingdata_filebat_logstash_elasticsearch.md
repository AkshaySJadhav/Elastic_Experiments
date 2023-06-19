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
  grok {
    match => { "message" => "\[%{SPACE}%{NUMBER:timestamp}\] %{WORD:service}\[%{NUMBER:pid}\]: %{GREEDYDATA:message}" }
}
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logstash-beat-pipeline"
  }
}
```

Once it is configure, we have to start the filebeat and logstash with below command:

```
root@osboxes:/usr/share/filebeat/bin# ./filebeat -e -c /etc/filebeat/filebeat.yml

{"log.level":"info","@timestamp":"2023-06-07T05:56:37.826-0400","log.origin":{"file.name":"instance/beat.go","file.line":724},"message":"Home path: [/usr/share/filebeat/bin] Config path: [/usr/share/filebeat/bin]
Data path: [/usr/share/filebeat/bin/data] Logs path: [/usr/share/filebeat/bin/logs]","service.name":"filebeat","ecs.version":"1.6.0"}
```

```
root@osboxes:~# /usr/share/logstash/bin/logstash -f /etc/logstash/logstash-beat-dmesg.yml
Using bundled JDK: /usr/share/logstash/jdk
```

```
osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/_cat/indices?v | grep logstash-beat-pipeline
yellow open   logstash-beat-pipeline                     CC89E3D7SbKrz1deXF1Irg   1   1        667            0    410.1kb        410.1kb
osboxes@osboxes:~$
```

Manually pushed the data to the dmesg file and could see that is being fetch by beats and filter by logstash. I was able to view the records after adding a new data view for the index.

```
root@osboxes:/home/osboxes# echo "[    5.455333] systemd[1]: Starting Load/Save Random Seed...[Akshay]" >> /var/log/dmesg
root@osboxes:/home/osboxes#
```

