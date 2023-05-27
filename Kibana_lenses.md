# Kibana Lens:- Metricbeat

Kibana Lens is an intuitive UI that helps you create data visualization with ease! Cabana Lens accomplishes this with a single visualization app where you can drag and drop the parameters
and change the visualization on the fly.

Make sure to install the lens same version as elasticsearch.

### 1. Installation: 
```
root@osboxes:/home/osboxes# apt-get install metricbeat=8.2.0
root@osboxes:/home/osboxes# systemctl start metricbeat
```

### 2. Perform some load testing so that we can visualize on kibana.

We are going to add some stress on the linux machine with "stress" utility.

```
root@osboxes:/home/osboxes# apt-get install stress

root@osboxes:/home/osboxes# stress --cpu 3 --timeout 120
stress: info: [2492] dispatching hogs: 3 cpu, 0 io, 0 vm, 0 hdd
stress: info: [2492] successful run completed in 126s

root@osboxes:/home/osboxes# stress --vm 5 --timeout 120
stress: info: [2534] dispatching hogs: 0 cpu, 0 io, 5 vm, 0 hdd
stress: info: [2534] successful run completed in 120s
root@osboxes:/home/osboxes#
```

### 3. Setup the Data View in Kibana.

<b>Step1:</b> Login to Kibana > Stack Management >> Data views >> Create a new Data view.
<per> 
  Name: Metricview
  Index: Metricview-*
</per>

<b>Step2:</b> Analytic >> Visualize Libaray >> New Viualization and select "Lens" and change aggergation to "Maximun" once we add the metric.

You will see the empty screen, now you have to select the metric from the left side to generate the graph.
1. system.process.cpu.total.percent --> What percent of CPU is being consumed.
2. process.executable --> what binary was are running.

Here is the screenshot of Data view:
![Alt text](https://github.com/AkshaySJadhav/Elastic_Experiments/blob/main/resources/Kibana.png)

Refer:<br>
[1]. https://www.elastic.co/guide/en/kibana/current/lens.html
