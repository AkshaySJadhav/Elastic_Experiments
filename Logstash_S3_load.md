
## Loading data from AWS S3 to the Elastic with Logstash.

### 1. Setup the S3 access.

Make sure that you have the read access over the S3 bucket and file preset. I am going to use the same access_log file which is available under the "elastic-experiment" bucket.
```
apple@Apples-MacBook-Pro Downloads % aws s3 ls s3://elastic-experiment
2023-05-26 00:06:53   23200421 access_log
apple@Apples-MacBook-Pro Downloads %
```
### 2. Create a new configuration.

The new configuration file must be created under /etc/logstash/conf.d/ directory. <br>
Here is the sample from my workstation:
```
root@osboxes:/usr/share/logstash/bin# cat /etc/logstash/conf.d/s3.conf
input {
	s3 {
		bucket => "elastic-experiment"
		access_key_id => "SSSSS"
		secret_access_key => "SSSS"
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
			hosts => "http://localhost:9200"
			index => "s3_data"}
  stdout { codec => rubydebug }
}
```

### 3. Start the logstash process.

You would start observing the verbose of actual data on the screen if connectivity is fine.

```
root@osboxes:/usr/share/logstash/bin# ./logstash -f /etc/logstash/conf.d/s3.conf
Using bundled JDK: /usr/share/logstash/jdk

[INFO ] 2023-05-25 14:48:43.858 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600, :ssl_enabled=>false}
[WARN ] 2023-05-25 14:48:57.901 [[main]-pipeline-manager] elasticsearch - Restored connection to ES instance {:url=>"http://localhost:9200/"}
[INFO ] 2023-05-25 14:49:02.073 [[main]-pipeline-manager] s3 - Registering {:bucket=>"elastic-experiment", :region=>"us-east-1"}
```

### 4. Verify if the indices are craeted.

Once Logstash has finished processing the data, you can verify its completion by checking the indices "s3_data".
```
root@osboxes:/usr/share/logstash/bin# curl -XGET 127.0.0.1:9200/_cat/indices?v
health status index                                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   demo-csv                                   Tsiz3zbBRmuzEUO0SQqN5Q   1   1          5            0     32.4kb         32.4kb
yellow open   movielens-sql                              q3DuQVFrT9e_1gIP9-lXFw   1   1       1682            0    360.5kb        360.5kb
yellow open   s3-log                                     _u2-sUJXRIans5Us-0GcoQ   1   1        500            0      3.6mb          3.6mb
root@osboxes:/usr/share/logstash/bin#
```
