## Load the CSV format Data to Elasticsearch via Logstash.


### 1. Download the sample csv data.
```
root@osboxes:~/csv-data# curl -O  https://raw.githubusercontent.com/coralogix-resources/elk-course-samples/master/csv-schema-short-numerical.csv
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   518  100   518    0     0    288      0  0:00:01  0:00:01 --:--:--   288
```

### 2. Create the configuration file to for CSV data under /etc/logstash/conf.d/
```
root@osboxes:/etc/logstash/conf.d# cat csv-read.conf
input {
  file {
    path => "/root/csv-data/csv-schema-short-numerical.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv {
      separator => ","
      skip_header => "true"
      columns => ["id","timestamp","paymentType","name","gender","ip_address","purpose","country","age"]
  }
}
output {
   elasticsearch {
     hosts => "http://localhost:9200"
     index => "demo-csv"
  }

stdout {}

}
root@osboxes:/etc/logstash/conf.d#
```

### 3. Start the logstash process and pass the configuration file.

```
root@osboxes:/usr/share/logstash/bin# ./logstash -f /etc/logstash/conf.d/csv-read.conf
Using bundled JDK: /usr/share/logstash/jdk
[INFO ] 2023-05-25 14:23:27.304 [main] runner - Starting Logstash {"logstash.version"=>"8.8.0", "jruby.version"=>"jruby 9.3.10.0 (2.6.8) 2023-02-01 107b2e6697 OpenJDK 64-Bit Server VM 17.0.7+7 on 17.0.7+7 +indy +jit [x86_64-linux]"}
```
### 4. Check the Indices.
Once Logstash has finished processing the data, you can verify its completion by checking the indices "demo-csv".
```
root@osboxes:/usr/share/logstash/bin# curl -XGET 127.0.0.1:9200/_cat/indices?v
health status index                                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   movies                                     5Roxm1-YRqeWobezK6huyQ   1   1          5            0      6.1kb          6.1kb
yellow open   demo-csv                                   Tsiz3zbBRmuzEUO0SQqN5Q   1   1          5            0     32.2kb         32.2kb
root@osboxes:/usr/share/logstash/bin#```
