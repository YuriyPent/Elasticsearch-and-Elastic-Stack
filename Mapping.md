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

***“star”:***
* ***unigram:  [ s, t, a, r ]***
* ***bigram:  [ st, ta, ar ]***
* ***trigram:  [ sta, tar ]***
* ***4-gram:  [ star ]***
***edge n-grams are built only on the beginning of each term.***
***
#### Indexing n-grams
```
curl -XPUT '127.0.0.1:9200/movies?pretty' -d '
{
    "settings": {
        "analysis": {
            "filter": {
                "autocomplete_filter": {
                    "type": "edge_ngram",
                        "min_gram": 1,
                            "max_gram": 20
                }
        },
    "analyzer": {
        "autocomplete": {
            "type": "custom",
                "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                            "autocomplete_filter"
                     ]
                 }
            }
        }
    }
}'
```
#### Now map your field with it
```
curl -XPUT '127.0.0.1:9200/movies/_mapping/movie?pretty' -d '
{
    "movie": {
        "properties" : {
            "title": {
                "type" : "string",
                    "analyzer": "autocomplete"
             } 
        }
    }
}'
```
#### But only use n-grams on the index side!
```
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match": {
            "title": {
                "query": "sta",
                    "analyzer": "standard"
            }
        }
    }
}'
```
***Otherwise our query will also get split into n-grams, and we’ll get results for everything that matches ‘s’, ‘t’, ‘a’, ‘st’, etc.***
***
#### Importing data
***you can import from just about anything stand-alone scripts can submit bulk documents via REST API logstash and beats can stream data from logs, S3, databases, and more AWS systems can stream in data via lambda or kinesis firehose kafka, spark, and more have Elasticsearch integration add-ons***
#### importing via script / json
* read in data from some distributed filesystem
* transform it into JSON bulk inserts
* submit via HTTP / REST to your elasticsearch cluster
```
import csv
import re
csvfile = open('ml-latest-small/movies.csv', 'r')
reader = csv.DictReader( csvfile )
for movie in reader:
    print ("{ \"create\" : { \"_index\": \"movies\", \"_type\": \"movie\", \"_id\" : \"" , movie['movieId'], "\" } }", sep='')
    title = re.sub(" \(.*\)$", "", re.sub('"','', movie['title']))
    year = movie['title'][-5:-1]
    if (not year.isdigit()):
        year = "2016"
    genres = movie['genres'].split('|')
    print ("{ \"id\": \"", movie['movieId'], "\", \"title\": \"", title, "\", \"year\":", year, ", \"genre\":[", end='', sep='')
    for genre in genres[:-1]:
        print("\"", genre, "\",", end='', sep='')
    print("\"", genres[-1], "\"", end = '', sep='')
    print ("] }")
```
#### importing via client api’s
***A less hacky script.***
***free elasticsearch client libraries are available for pretty much any language.***
* java has a client maintained by elastic.co
* python has an elasticsearch package
* elasticsearch-ruby
* several choices for scala
* elasticsearch.pm module for perl
***You don’t have to wrangle JSON.**
```
es = elasticsearch.Elasticsearch()
es.indices.delete(index="ratings",ignore=404)
deque(helpers.parallel_bulk(es,readRatings(),index="ratings",doc_type
es.indices.refresh()
```
***For completeness:***
```
import csv from collections import deque
import elasticsearch from elasticsearch import helpers

def readMovies():

    csvfile = open('ml-latest-small/movies.csv', 'r')
    reader = csv.DictReader( csvfile )
    titleLookup = {}
    for movie in reader:
    titleLookup[movie['movieId']] = movie['title']
    return titleLookup

def readRatings():
    csvfile = open('ml-latest-small/ratings.csv', 'r')
    titleLookup = readMovies()
    reader = csv.DictReader( csvfile )
        for line in reader:
            rating = {}
            rating['user_id'] = int(line['userId'])
            rating['movie_id'] = int(line['movieId'])
            rating['title'] = titleLookup[line['movieId']]
            rating['rating'] = float(line['rating'])
            rating['timestamp'] = int(line['timestamp'])
            yield rating
            
es = elasticsearch.Elasticsearch()

es.indices.delete(index="ratings",ignore=404)
deque(helpers.parallel_bulk(es,readRatings(),index="ratings",doc_type="rating"), maxlen=0)
es.indices.refresh()
```
***
#### Installing logstash
***sudo apt-get update***
***sudo apt-get install logstash***
***sudo vi /etc/logstash/conf.d/logstash.conf***
```
input {
    file {
        path => "/home/fkane/access_log“
        start_position => "beginning"
        ignore_older => 0
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
date {
    match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
}
output {
    elasticsearch {
        hosts => ["localhost:9200"]
    }
stdout {
    codec => rubydebug
    }
}
```

***cd /usr/share/logstash/***

***sudo bin/logstash -f /etc/logstash/conf.d/logstash.conf***

#### Logstash with mysql

***get a mysql connector from https://dev.mysql.com/downloads/connector/j/***

***wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.42.zip***

***unzip mysql-connector-java-5.1.42.zip***
```
input {
    jdbc {
        jdbc_connection_string => "jdbc:mysql://localhost:3306/movielens"
        jdbc_user => "root"
        jdbc_password => “password"
        jdbc_driver_library => "/home/fkane/mysql-connector-java-5.1.42/mysql-connector-java-5.1.42-bin.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        statement => "SELECT * FROM movies"
    }
}
```
#### Logstash with s3 (amazon web services’ simple storage service cloud-based distributed storage system)
***integration is easy-peasy.***
```
input {
    s3 {
        bucket => "sundog-es"
        access_key_id => "AKIAIS****C26Y***Q"
        secret_access_key => "d*****FENOXcCuNC4iTbSLbibA*****eyn****"
    }
}
```
#### Logstash with kafka
apache kafka
* open-source stream processing platform
* high throughput, low latency
* publish/subscribe
* process streams
* store streams
```
input {
    kafka {
        bootstrap_servers => "localhost:9092"
        topics => ["kafka-logs"]
    }
}
```
***
#### Elasticsearch with spark
***What is apache spark

* “a fast and general engine for large-scale data processing”
* a faster alternative to mapreduce
* spark applications are written in java, scala, python, or r
* supports sql, streaming, machine learning, and graph processing
**flink is nipping at spark’s heels, and can also integrate with elasticsearch.**
```
./spark-2.1.1-bin-hadoop2.7/bin/spark-shell --packages org.elasticsearch:elasticsearch-spark-20_2.11:5.4.3

import org.elasticsearch.spark.sql._

case class Person(ID:Int, name:String, age:Int, numFriends:Int)

def mapper(line:String): Person = {
    val fields = line.split(',')
    val person:Person = Person(fields(0).toInt, fields(1), fields(2).toInt, fields(3).toInt)
    return person
}

import spark.implicits._

val lines = spark.sparkContext.textFile("fakefriends.csv")
val people = lines.map(mapper).toDF()

people.saveToEs("spark/people")
```
***
#### Aggregations
***Bucket by rating value:***
```
curl -XGET 
'127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
    "aggs": {
        "ratings": {
            "terms": {
                "field": "rating"
            }
        }
    }
}'
```
#### Histograms
***display ratings by 1.0-rating intervals***
```
curl -XGET 
'127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
    "aggs" : {
        "whole_ratings": {
            "histogram": {
                "field": "rating",
                    "interval": 1.0
                }
           }
      }
}'
```
***Count up movies from each decade***
```
curl -XGET 
'127.0.0.1:9200/movies/movie/_search?size=0&pretty' -d ‘
{
    "aggs" : {
        "release": {
            "histogram": {
                "field": "year",
                    "interval": 10
                }
           }
      }
}
```
***
#### Time series
***Break down website hits by hour:***
```
curl -XGET '127.0.0.1:9200/logstash-2015.12.04/logs/_search?size=0&pretty' -d ‘
{
    "aggs" : {
        "timestamp": {
            "date_histogram": {
                "field": "@timestamp",
                    "interval": "hour"
                }
          }
     }
}'
```
***When does google scrape me?***
```
curl -XGET '127.0.0.1:9200/logstash-2015.12.04/logs/_search?size=0&pretty' -d ‘
{
    "query" : {
        "match": {
            "agent": "Googlebot"
        }
    },
    "aggs" : {
        “timestamp": {
            "date_histogram": {
                "field": "@timestamp",
                    "interval": "hour"
            }
        }
    }
}
```
***
#### Nested aggregations
***For reference, here’s the final query***
```
curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d ‘
{
    "query": {
        "match_phrase": {
            "title": "Star Wars"
        }
    },
    "aggs" : {
        "titles": {
            "terms": {
                "field": "title.raw"
    },
    "aggs": {
        "avg_rating": {
            "avg": {
                "field" : "rating"
                        }
                   }
              }
         }
    }
}'
```
***
#### Using Kibana
***Installing kibana***

* *sudo apt-get install kibana*
* *sudo vi /etc/kibana/kibana.yml*
* *change server.host to 0.0.0.0*
* *add xpack.security.enabled: false*
* *sudo /bin/systemctl daemon-reload*
* *sudo /bin/systemctl enable kibana.service*
* *sudo /bin/systemctl start kibana.service*
* *kibana is now available on port 5601*
