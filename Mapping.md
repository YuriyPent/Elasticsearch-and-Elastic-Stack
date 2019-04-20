#### What is a mapping?

A mapping is a schema definition.

Elasticsearch has reasonable defaults, but sometimes you need to customize them.
```
curl -XPUT 127.0.0.1:9200/movies -d '
{
    "mappings": {
    "movie": {
    "_all": {"enabled": false},
    "properties" : {
    "year" : {“type": "date"}
        }
      }
    }
}'
```
***
#### Insert
```
curl -XPUT 127.0.0.1:9200/movies/movie/109487 -d '

{
  "genre" : ["IMAX","Sci-Fi"],
  "title" : "Interstellar",
  "year" : 2014
}'
```
***
#### Json bulk import
```
curl -XPUT 127.0.0.1:9200/_bulk –d ‘
{ 
  "create" : { 
    "_index" : "movies", "_type" : "movie", "_id" : "135569" } }
{ "id": "135569", "title" : "Star Trek Beyond", "year":2016 , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "122886" }}
{ "id": "122886", "title" : "Star Wars: Episode VII - The Force Awakens", "year":2015 , "genre":["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"]}
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "109487" } }
{ "id": "109487", "title" : "Interstellar", "year":2014 , "genre":["Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "58559" } }
{ "id": "58559", "title" : "Dark Knight, The", "year":2008 , "genre":["Action", "Crime", "Drama", "IMAX"] }
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "1924" }}
{ "id": "1924", "title" : "Plan 9 from Outer Space", "year":1959 , "genre":["Horror", "Sci-Fi"] 
} ‘
```
***
#### Partial update api
```
curl -XPOST 127.0.0.1:9200/movies/movie/109487/_update -d '
{
    "doc": {
  "title": "Interstellar"
    }
}'
```
***
#### Delete document
```
curl -XDELETE 127.0.0.1:9200/movies/movie/58559
```
***
#### Request body search
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
  "query": {
    "match": {
    "title": "star"
    }
  }
}'
```
***
#### Example: boolean query with a filter
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
  "query":{
    "bool": {
      "must": {"term": {"title": "trek"}},
        "filter": {"range": {"year": {"gte": 2010}}}
        }
     }
}'
```
#### Some types of filters

term: filter by exact values {“term”: {“year”: 2014}}

terms: match if any exact values in a list match {“terms”: {“genre”: [“Sci-Fi”, “Adventure”] } }

range: Find numbers or dates in a given range (gt, gte, lt, lte) {“range”: {“year”: {“gte”: 2010}}}

exists: Find documents where a field exists {“exists”: {“field”: “tags”}}

missing: Find documents where a field is missing {“missing”: {“field”: “tags”}}

bool: Combine filters with Boolean logic (must, must_not, should)

match_all: returns all documents and is the default. Normally used with a filter.{“match_all”: {}}

match: searches analyzed results, such as full text search.{“match”: {“title”: “star”}}

multi_match: run the same query on multiple fields.{“multi_match”: {“query”: “star”, “fields”: [“title”, “synopsis” ] } }

bool: Works like a bool filter, but results are scored by relevance.
***
#### Syntax reminder

queries are wrapped in a “query”: { } block, filters are wrapped in a “filter”: { } block.

you can combine filters inside queries, or queries inside filters too.
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
  "query":{
    "bool": {
      "must": {"term": {"title": "trek"}},
        "filter": {"range": {"year": {"gte": 2010}}}
          }
      }
}'
```

