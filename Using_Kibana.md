# Installation of Kibana

### 1. Install the package.


Make sure that elasticsearch version matches with the kibana version. You can check the elastic search version with curl -XGET localhost:9200 command.
```
root@osboxes:~/kafka_2.12-2.7.0/bin#apt-get install kibana=8.7.0
root@osboxes:~/kafka_2.12-2.7.0/bin# vim /etc/kibana/kibana.yml
Changed server.host: "0.0.0.0" to server.host: "0.0.0.0"
````

### 2. Start the Kibana service.

It'll probably take a few minutes for that to spin up but eventually that means Cabana will be available on Port 50601 for us.
```
root@osboxes:~# systemctl daemon-reload
root@osboxes:~# systemctl start kibana
root@osboxes:~# systemctl enable kibana.service
```
