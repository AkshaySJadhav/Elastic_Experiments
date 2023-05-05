
## Concurrency issue with documents in Elasticsearch.


Concurrency issue occures when two users are trying update the same document at the same time. To avoid this we can use sequance number and primay no in ther rest API.


Following document has the primary term and the sequence no:
```
osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/movies/_doc/109487?pretty
{
  "_index" : "movies",
  "_id" : "109487",
  "_version" : 3,
  "_seq_no" : 6,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "genre" : [
      "IMAX",
      "Sci-Fi"
    ],
    "title" : "Interstellar",
    "year" : 2014
  }
}
```

This is how we suppose to update the document:
```
osboxes@osboxes:~$ curl -XPUT "127.0.0.1:9200/movies/_doc/109487?if_seq_no=6&if_primary_term=1" -d '
{
"genre" : ["IMAX", "Sci-Fi"],
"title" : "Interstellar Foo",
"year" : 2014
}'
osboxes@osboxes:~$
{"_index":"movies","_id":"109487","_version":4,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":12,"_primary_term":1}
```


Let's run the same command again and see if that works? No, it failed becasue the secuence has been updated.

```
osboxes@osboxes:~$ curl -XPUT "127.0.0.1:9200/movies/_doc/109487?if_sql_no=6&id_primary_term=1" -d '
{
"genre" : ["IMAX", "Sci-Fi"],
"title" : "Interstellar Foo",
"year" : 2014
}'
{"error":{"root_cause":[{"type":"illegal_argument_exception","reason":"request [/movies/_doc/109487] contains unrecognized parameters: [id_primary_term] -> did you mean [if_primary_term]?, [if_sql_no] -> did you mean [if_seq_no]?"}],"type":"illegal_argument_exception","reason":"request [/movies/_doc/109487] contains unrecognized parameters: [id_primary_term] -> did you mean [if_primary_term]?, [if_sql_no] -> did you mean [if_seq_no]?"},"status":400}
```

OR, you can use the retry_on_conflict option which simply means how many times should the operation be retried when a conflict occurs.

```
osboxes@osboxes:~$ curl -XPOST "127.0.0.1:9200/movies/_update/109487?retry_on_conflict=5" -d '
{
"doc": {
"title": "Interstellar Typooo"
}
}'
osboxes@osboxes:~$
{"_index":"movies","_id":"109487","_version":5,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":13,"_primary_term":1}
```

Resource: https://elasticsearch-py.readthedocs.io/en/7.x/api.html
