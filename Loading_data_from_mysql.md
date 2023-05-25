## Loading data to Elasticsearch from MYSQL via Logstash.


### 1. Install latest version of MySQL.

```
root@osboxes:~# apt-get install mysql-server
```

### 2. Download the sample data and JDBC driver.

In order to connect to the mysql, we need to download the driver file from the mysql website. We are going to using the file mysql-connector-java-8.0.30.jar file. 

```
root@osboxes:~# wget http://files.grouplens.org/datasets/movielens/ml-100k.zip
--2023-05-25 12:14:53--  http://files.grouplens.org/datasets/movielens/ml-100k.zip
Resolving files.grouplens.org (files.grouplens.org)... 128.101.65.152
Connecting to files.grouplens.org (files.grouplens.org)|128.101.65.152|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4924029 (4.7M) [application/zip]
Saving to: ‘ml-100k.zip’

ml-100k.zip                              100%[================================================================================>]   4.70M   142KB/s    in 31s

2023-05-25 12:15:28 (154 KB/s) - ‘ml-100k.zip’ saved [4924029/4924029]

root@osboxes:~#
```

### 3. Create the database and import the data.

```
root@osboxes:~# sudo mysql --local-infile=1 -u root -p
Enter password:
mysql> create database  movielense;
Query OK, 1 row affected (4.66 sec)

mysql> CREATE TABLE movielense.movies(
    -> movieID INT PRIMARY KEY NOT NULL,
    -> title TEXT,
    -> releaseDate DATE
    -> );
Query OK, 0 rows affected (6.47 sec)

mysql> LOAD DATA LOCAL INFILE 'u.item' INTO TABLE movielense.movies CHARACTER SET latin1 FIELDS TERMINATED BY '|'
    -> (movieID, title, @var3)
    -> set releaseDate = STR_TO_DATE(@var3, '%d-%M-%Y');
Query OK, 1682 rows affected, 1683 warnings (5.18 sec)
Records: 1682  Deleted: 0  Skipped: 0  Warnings: 1683

mysql>
```


### 4. Create a new configuration file for the Mysql.

Please make sure that that jar is available in the location that has been mentioned in the configuration file.
```
root@osboxes:~# cat /etc/logstash/conf.d/mysql.conf
input {
    jdbc {
        jdbc_driver_library => "/root/mysql-connector-java-8.0.30.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://localhost:3306/movielense"
        jdbc_user => "root"
        jdbc_password => "password"
        statement => "SELECT * FROM movies"
    }
}
output {
   stdout {codec => json_lines}
   elasticsearch {
        hosts => "http://127.0.0.1:9200"
        index => "movielens-sql"
        }
}
root@osboxes:~#
```

### 5. Start the logstash with mysql configuration.

You will start observing database verbose on the screens in sometime.
```
root@osboxes:/usr/share/logstash/bin# ./logstash -f /etc/logstash/conf.d/mysql.conf
Using bundled JDK: /usr/share/logstash/jdk

{"releasedate":"1997-01-01T05:00:00.000Z","title":"Red Corner (1997)","movieid":754,"@timestamp":"2023-05-25T17:07:19.127126203Z","@version":"1"}
{"releasedate":"1995-01-01T05:00:00.000Z","title":"Jumanji (1995)","movieid":755,"@timestamp":"2023-05-25T17:07:19.127508606Z","@version":"1"}
```

### 6. Check the Indices.
Once Logstash has finished processing the data, you can verify its completion by checking the indices "movielens-sql".
```
root@osboxes:/usr/share/logstash/bin# curl -XGET 127.0.0.1:9200/_cat/indices?v
health status index                                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   movies                                     5Roxm1-YRqeWobezK6huyQ   1   1          5            0      6.1kb          6.1kb
yellow open   movielens-sql                              q3DuQVFrT9e_1gIP9-lXFw   1   1       1682            0    359.7kb        359.7kb
yellow open   series                                     lz1awNsPQ3GInc5U3fSraw   1   1          8            0     10.2kb         10.2kb
yellow open   demo-flattened                             NnS4YmqgQR-PEocLZhvIRA   1   1          1            0     19.4kb         19.4kb
yellow open   .ds-logs-generic-default-2023.05.25-000001 smLN5smCQFylV-HO3yFVXQ   1   1     102972            0     24.3mb         24.3mb
yellow open   shakespeare                                K5ZC2rvnQg6avpilxVysTg   1   1     111396            0     17.6mb         17.6mb
yellow open   microservice-logs                          WM2ZOxbXTpuNXkwpeoP9FA   1   1          3            0     12.2kb         12.2kb
yellow open   demo-default                               bjqO7jvmTcyIsamPT5B6wA   1   1          1            0      8.9kb          8.9kb
root@osboxes:/usr/share/logstash/bin#
```
