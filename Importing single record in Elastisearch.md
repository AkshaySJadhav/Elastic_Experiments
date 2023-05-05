
## Importing the single JSON data into the Elastic search.


We are going to import the movie "Interstellar" and information associte with it.   
```
osboxes@osboxes:~$ curl -XPUT 127.0.0.1:9200/movies -d '

> {
    "mappings":{
      "properties": {
               "year": {
                  "type": "date"
                        }
                    }
               }
> }'
```

This is the acknowledgment received from server:
```
osboxes@osboxes:~$ {"acknowledged":true,"shards_acknowledged":true,"index":"movies"}
```

Looks like the mapping is part of the index now:
```
osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/movies/_mapping
{"movies":{"mappings":{"properties":{"year":{"type":"date"}}}}}
```

Inserting a moving into the elastisearch index:
```
osboxes@osboxes:~$ curl -XPUT 127.0.0.1:9200/movies/_doc/109487?pretty -d '
> {
> "genre": ["IMAX", "Sci-Fi"],
> "title" : "Interstellar",
> "year" : 2014
> }'
{
  "_index" : "movies",
  "_id" : "109487",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
osboxes@osboxes:~$

osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/movies/_search?pretty
{
  "took" : 5204,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "109487",
        "_score" : 1.0,
        "_source" : {
          "genre" : [
            "IMAX",
            "Sci-Fi"
          ],
          "title" : "Interstellar",
          "year" : 2014
        }
      }
    ]
  }
}
osboxes@osboxes:~$
```
