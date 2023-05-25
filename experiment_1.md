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
	fike {
		path => "/hom/student/access_log"
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

### 3. 
