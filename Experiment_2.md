### Parent Child relations with Start War movies. 🎥  


So let's go ahead and set up a parent child mapping in Elasticsearch to associate films with the franchise they came from.

And we'll do that with our Star Wars movies here. So what we're going to do is set up a new index called Series that this data will live within.


So the first thing we need to do is create this new index with that relation as part of its mapping.

Here's what that syntax would look like.
```
osboxes@osboxes:~$ curl -XPUT 127.0.0.1:9200/series -d '
{
  "mappings": {
    "properties": {
      "film_to_franchise": {
          "type": "join",
            "relations": {"franchise" : "film"}
                            }
                  }
              }
}'

osboxes@osboxes:~$ {"acknowledged":true,"shards_acknowledged":true,"index":"series"}
```

Now 

```
osboxes@osboxes:~$ wget http://media.sundog-soft.com/es8/series.json
--2023-05-06 03:49:02--  http://media.sundog-soft.com/es8/series.json
Resolving media.sundog-soft.com (media.sundog-soft.com)... 54.231.231.65, 54.231.169.193, 52.217.172.57, ...
Connecting to media.sundog-soft.com (media.sundog-soft.com)|54.231.231.65|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1979 (1.9K) [application/octet-stream]
Saving to: ‘series.json’

series.json                                  100%[============================================>]   1.93K  --.-KB/s    in 0s

2023-05-06 03:49:03 (196 MB/s) - ‘series.json’ saved [1979/1979]

```` 

osboxes@osboxes:~$ curl -XPUT 127.0.0.1:9200/series/_bulk?pretty --data-binary @series.json
