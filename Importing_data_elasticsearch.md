
## Importing JSON data into the Elastic search and performing operations.

### Inserting the document:


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

This is the acknowledgement received from server:
```
osboxes@osboxes:~$ {"acknowledged":true,"shards_acknowledged":true,"index":"movies"}
```

Looks like the mapping is part of the index now:
```
osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/movies/_mapping
{"movies":{"mappings":{"properties":{"year":{"type":"date"}}}}}
```

Inserting a moving into the elastisearch index. 
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

To bulk load we can use the following command. There is a conflict error as the document was already uploaded to elastic.

```
curl -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @movies.json
{
  "took" : 401,
  "errors" : true,
  "items" : [
    {
      "create" : {
        "_index" : "movies",
        "_id" : "135569",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : "movies",
        "_id" : "122886",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : "movies",
        "_id" : "109487",
        "status" : 409,
        "error" : {
          "type" : "version_conflict_engine_exception",
          "reason" : "[109487]: version conflict, document already exists (current version [1])",
          "index_uuid" : "TUnRSHRrQw6gU5IbxgAqbg",
          "shard" : "0",
          "index" : "movies"
        }
      }
    },
    {
      "create" : {
        "_index" : "movies",
        "_id" : "58559",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : "movies",
        "_id" : "1924",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 4,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```



### Updating the document:

Let's try updating the document.

Elasticsearch documents are immutable and when you upadate the document, new document will be created with an increment_version and old marks for deletion.
 

```
osboxes@osboxes:~$ curl -XPUT 127.0.0.1:9200/movies/_doc/109487?pretty -d '
> {
> "genre": ["IMAX", "Sci-Fi"],
> "title": "interstellar foo",
> "year": 2014
> }'
{
  "_index" : "movies",
  "_id" : "109487",
  "_version" : 2,    ===>How many time this document has been updated.
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 1
}
osboxes@osboxes:~$
```

### Deleting the document:

Fetching the document ID: I am searching Dark Knight movie to delete. 
```
osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/movies/_search?q=Dark
{"took":10367,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":1.5169399,"hits":[{"_index":"movies","_id":"58559","_score":1.5169399,"_source":{ "id": "58559", "title" : "Dark Knight, The", "year":2008 , "genre":["Action", "Crime", "Drama", "IMAX"] }}]}}
```

Issue a delete curl command":
```
osboxes@osboxes:~$ curl -XDELETE 127.0.0.1:9200/movies/_doc/58559?pretty
{
  "_index" : "movies",
  "_id" : "58559",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 7,
  "_primary_term" : 1
}
osboxes@osboxes:
```



