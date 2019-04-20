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

**term:**
*filter by exact values* 
```{“term”: {“year”: 2014}}```

**terms:** 
*match if any exact values in a list match* 
```{“terms”: {“genre”: [“Sci-Fi”, “Adventure”] } }```

**range:** 
*Find numbers or dates in a given range (gt, gte, lt, lte)*
```{“range”: {“year”: {“gte”: 2010}}}```

**exists:** 
*Find documents where a field exists* 
```{“exists”: {“field”: “tags”}}```

**missing:** 
*Find documents where a field is missing* 
```{“missing”: {“field”: “tags”}}```

**bool:** 
*Combine filters with Boolean logic (must, must_not, should)*

**match_all:** 
*returns all documents and is the default. Normally used with a filter.*
``{“match_all”: {}}``

**match:** 
*searches analyzed results, such as full text search.*
``{“match”: {“title”: “star”}}``

**multi_match:** 
*run the same query on multiple fields.*
``{“multi_match”: {“query”: “star”, “fields”: [“title”, “synopsis” ] } }``

**bool:** 
*Works like a bool filter, but results are scored by relevance.*
***
#### Syntax reminder

**queries are wrapped in a “query”: { } block, 
filters are wrapped in a “filter”: { } block.**

*You can combine filters inside queries, or queries inside filters too.*
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
***
#### Phrase matching
**Must find all terms, in the right order.**
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match_phrase": {
            "title": "star wars"
        }
    }
}'
```
**order matters, but you’re OK with some words being in between the terms:**
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match_phrase": {
            "title": {"query": "star beyond", "slop": 1}
        }
    }
}'
```
***The slop represents how far you’re willing to let a term move to satisfy a phrase (in either direction!).
Another example: “quick brown fox” would match “quick fox” with a slop of 1.***

#### Proximity queries
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match_phrase": {
            "title": {"query": "star beyond", "slop": 100}
        }
    }
}'
```
***Remember this is a query – results are sorted by relevance. Just use a really high slop if you want to get any documents that contain the words in your phrase, but want documents that have the words closer together scored higher.***
***
#### pagination syntax
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?size=2&from=2&pretty'

curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
    "from": 2,
        "size": 2,
            "query": {"match": {"genre": "Sci-Fi"}}
}'
```
***
#### Sorting your results is usually quite simple.
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?sort=year&pretty'
```
***A string field that is analyzed for full-text search can’t be used to sort documents. This is because it exists in the inverted index as individual terms, not as the entire string.***
***If you need to sort on an analyzed field, map a not_analyzed copy.***
```
curl -XPUT 127.0.0.1:9200/movies/ -d '
{
    "mappings": {
        "movie": {
            "_all": {"enabled": false},
                "properties" : {
                    "title": {
                        "type" : "string",
                            "fields": {
                                  "raw": {
                                     "type": "string",
                                        "index": "not_analyzed"
                        }
                    } 
                }
            }
        }
    }
}'
```
***Now you can sort on the not_analyzed “raw” field.***
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?sort=title.raw&pretty'
```
***Sadly, you cannot change the mapping on an existing index. You’d have to delete it, set up a new mapping, and re-index it. Like the number of shards, this is something you should think about before importing data into your index.***

#### Another filtered query
```
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d'
{
    "query":{
        "bool": {
            "must": {"match": {"genre": "Sci-Fi"}},
                "must_not": {"match": {"title": "trek"}},
                    "filter": {"range": {"year": {"gte": 2010, "lt": 2015}}}
           }
      }
}'
```
#### Fuzzy matches

***A way to account for typos and misspellings the levenshtein edit distance accounts for:***
* ***substitutions of characters (interstellar -> intersteller)***
* ***insertions of characters (interstellar -> insterstellar)***
* ***deletion of characters (interstellar -> interstelar) all of the above have an edit distance of 1.***

#### The fuzziness parameter
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "fuzzy": {
            "title": {"value": "intrsteller", "fuzziness": 2}
        }
    }
}'
```
***
#### Partial matching
***Prefix queries on strings. 
If we remapped year to be a string…***
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
    "query": {
        "prefix": {
            "year": "201"
        }
    }
}'
```
***wildcard queries***
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
"query": {
"wildcard": {
"year": "1*"
}
}
}'
```
***“regexp” queries also exist.***
#### query-time search-as-you-type
```
curl -XGET '127.0.0.1:9200/movies/movie/_search?pretty' -d '
{
    "query": {
        "match_phrase_prefix": {
            "title": {
                "query": "star trek",
                    "slop": 10
            }
        }
    }
}'
```
#### index-time with N-grams

***“star”:
unigram:  [ s, t, a, r ]
bigram:  [ st, ta, ar ]
trigram:  [ sta, tar ]
4-gram:  [ star ]
edge n-grams are built only on the beginning of each term.***
