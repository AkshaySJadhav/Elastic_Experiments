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

Now load the star wars series json data to the elastic. 

```
osboxes@osboxes:~$ wget https://github.com/AkshaySJadhav/Elastic_Experiments/blob/main/resources/series.json
HTTP request sent, awaiting response... 200 OK
Length: 1979 (1.9K) [application/octet-stream]
Saving to: ‘series.json’

series.json                                  100%[============================================>]   1.93K  --.-KB/s    in 0s

2023-05-06 03:49:03 (196 MB/s) - ‘series.json’ saved [1979/1979]

osboxes@osboxes:~$ curl -XPUT 127.0.0.1:9200/series/_bulk?pretty --data-binary @series.json
```` 


Let's access the movies that has the parent as "Star Wars"

```
osboxes@osboxes:~$ curl -XGET 127.0.0.1:9200/series/_search?pretty -d '
> {
> "query":  {
> "has_parent": {
> "parent_type": "franchise",
> "query":{
> "match": {
> "title" : "Star Wars"
> }
> }
> }
> }
> }'
{
  "took" : 403,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 7,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "series",
        "_id" : "260",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "260",
          "film_to_franchise" : {
            "name" : "film",
            "parent" : "1"
          },
          "title" : "Star Wars: Episode IV - A New Hope",
          "year" : "1977",
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "series",
        "_id" : "1196",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "1196",
          "film_to_franchise" : {
            "name" : "film",
            "parent" : "1"
          },
          "title" : "Star Wars: Episode V - The Empire Strikes Back",
          "year" : "1980",
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "series",
        "_id" : "1210",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "1210",
          "film_to_franchise" : {
            "name" : "film",
            "parent" : "1"
          },
          "title" : "Star Wars: Episode VI - Return of the Jedi",
          "year" : "1983",
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "series",
        "_id" : "2628",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "2628",
          "film_to_franchise" : {
            "name" : "film",
            "parent" : "1"
          },
          "title" : "Star Wars: Episode I - The Phantom Menace",
          "year" : "1999",
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "series",
        "_id" : "5378",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "5378",
          "film_to_franchise" : {
            "name" : "film",
            "parent" : "1"
          },
          "title" : "Star Wars: Episode II - Attack of the Clones",
          "year" : "2002",
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi",
            "IMAX"
          ]
        }
      },
      {
        "_index" : "series",
        "_id" : "33493",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "33493",
          "film_to_franchise" : {
            "name" : "film",
            "parent" : "1"
          },
          "title" : "Star Wars: Episode III - Revenge of the Sith",
          "year" : "2005",
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "series",
        "_id" : "122886",
        "_score" : 1.0,
        "_routing" : "1",
        "_source" : {
          "id" : "122886",
          "film_to_franchise" : {
            "name" : "film",
            "parent" : "1"
          },
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : "2015",
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}
osboxes@osboxes:~$
```
