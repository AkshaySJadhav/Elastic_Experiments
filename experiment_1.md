# Installing Logstash on Ubuntu and importing data.


### 1. Install the package on the linux machine:

```
#sudo apt-get update
#sudo apt-get install logstash
```

### 2. We need to specify what input and output plugins we are going to use in the configuration file. 
Let me walk you through the configuration file:
1. Input: This is where we specify the input data and reading behaviour.
2. Filter: We have a filter block that configure log slash to know how to extract structure data out of this Apache access log file.
3. Output: We'll send it to our local Elasticsearch server here, running on our local host on Port 9200. <br>
Ref: https://www.elastic.co/guide/en/logstash/current/config-examples.html


```
#vim /etc/logstash/conf.d/logstash.conf
input {
	file {
		path => "/root/access_log"
		start_position => "beginning"
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
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

### 3. Download the sample data to load into logstash.

- Download the access_log file from the resources.
- Move the downloaded file to the /root folder.
- Load the access_log file into Logstash.

With these steps, the configuration part is now completed.
```
root@osboxes:~# wget https://github.com/AkshaySJadhav/Elastic_Experiments/blob/main/resources/access_log
--2023-05-25 09:56:04--  https://github.com/AkshaySJadhav/Elastic_Experiments/blob/main/resources/access_log
Resolving github.com (github.com)... 20.207.73.82
Connecting to github.com (github.com)|20.207.73.82|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘access_log’

access_log.1            [ <=>                ] 137.87K  --.-KB/s    in 0.07s

2023-05-25 09:56:05 (2.02 MB/s) - ‘access_log’ saved [141179]

root@osboxes:~#
```
### 4. Start the Logstash process.
Navigate to home folder of logstash and pass the configuration file to the start script. 

```
root@osboxes:/usr/share/logstash/bin# ./logstash -f /etc/logstash/conf.d/logstash.conf
Using bundled JDK: /usr/share/logstash/jdk
```
<br> It will take another minute or so to really start processing that data. Now, there is a lot of data in that log file, so we have to wait for it to get through it all.

### 5. Review all the indices.

Once Logstash has finished processing the data, you can verify its completion by checking the uploaded file ".ds-logs-generic-default-2023.05.25-000001". This file should have the same size as the original "access_log" file.

``` 
root@osboxes:~# curl -XGET 127.0.0.1:9200/_cat/indices?v
health status index                                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   movies                                     5Roxm1-YRqeWobezK6huyQ   1   1          5            0      6.1kb          6.1kb
yellow open   series                                     lz1awNsPQ3GInc5U3fSraw   1   1          8            0     10.2kb         10.2kb
yellow open   demo-flattened                             NnS4YmqgQR-PEocLZhvIRA   1   1          1            0     19.4kb         19.4kb
yellow open   .ds-logs-generic-default-2023.05.25-000001 smLN5smCQFylV-HO3yFVXQ   1   1     102972            0     24.3mb         24.3mb
yellow open   shakespeare                                K5ZC2rvnQg6avpilxVysTg   1   1     111396            0     17.6mb         17.6mb
yellow open   microservice-logs                          WM2ZOxbXTpuNXkwpeoP9FA   1   1          3            0     12.2kb         12.2kb
yellow open   demo-default                               bjqO7jvmTcyIsamPT5B6wA   1   1          1            0      8.9kb          8.9kb
root@osboxes:~#
```

You can execute the search command on elasticsearch indices and confirmed the origin of the data.

```
root@osboxes:~# curl -XGET '127.0.0.1:9200/.ds-logs-generic-default-2023.05.25-000001/_search?pretty'
>
 },
          "log" : {
            "file" : {
              "path" : "/root/access_log"
 }
```

[1]. https://www.elastic.co/logstash/ <br>
[2]. https://www.elastic.co/blog/a-practical-introduction-to-logstash
