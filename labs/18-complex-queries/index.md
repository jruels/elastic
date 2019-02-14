# Elastic Stack Lab 18

In this lab we will be using some more advanced techniques to query our data. 

**NOTE:** The following commands will be run in the Kibana console. 

## Setup
First let's start by creating a new index for our books and specifying it should have 1 shard
```
PUT /bookdb_index
    { "settings": { "number_of_shards": 1 }}
``` 

Now we need to insert some data
```
POST /bookdb_index/book/_bulk
    { "index": { "_id": 1 }}
    { "title": "Elasticsearch: The Definitive Guide", "authors": ["clinton gormley", "zachary tong"], "summary" : "A distibuted real-time search and analytics engine", "publish_date" : "2015-02-07", "num_reviews": 20, "publisher": "oreilly" }
    { "index": { "_id": 2 }}
    { "title": "Taming Text: How to Find, Organize, and Manipulate It", "authors": ["grant ingersoll", "thomas morton", "drew farris"], "summary" : "organize text using approaches such as full-text search, proper name recognition, clustering, tagging, information extraction, and summarization", "publish_date" : "2013-01-24", "num_reviews": 12, "publisher": "manning" }
    { "index": { "_id": 3 }}
    { "title": "Elasticsearch in Action", "authors": ["radu gheorge", "matthew lee hinman", "roy russo"], "summary" : "build scalable search applications using Elasticsearch without having to do complex low-level programming or understand advanced data science algorithms", "publish_date" : "2015-12-03", "num_reviews": 18, "publisher": "manning" }
    { "index": { "_id": 4 }}
    { "title": "Solr in Action", "authors": ["trey grainger", "timothy potter"], "summary" : "Comprehensive guide to implementing a scalable search engine using Apache Solr", "publish_date" : "2014-04-05", "num_reviews": 23, "publisher": "manning" }
```

## Boost
Since we are searching across multiple fields, we may want to boost the scores in a certain field. In the contrived example below, we boost scores from the summary field by a factor of 3 in order to increase the importance of the summary field, which will, in turn, increase the relevance of document`_id 4`.
```
POST /bookdb_index/book/_search
{
    "query": {
        "multi_match" : {
            "query" : "elasticsearch guide",
            "fields": ["title", "summary^3"]
        }
    },
    "_source": ["title", "summary", "publish_date"]
}
```

## Bool Query
The AND_OR_NOT operators can be used to fine tune our search queries in order to provide more relevant or specific results. This is implemented in the search API as a bool query. The bool query accepts a must parameter (equivalent to AND), a must_not parameter (equivalent to NOT), and a should parameter (equivalent to OR). For example, if I want to search for a book with the word “Elasticsearch” OR “Solr” in the title, AND is authored by “clinton gormley” but NOT authored by “radu gheorge”:

```
POST /bookdb_index/book/_search
{
  "query": {
    "bool": {
      "must": {
        "bool" : { 
          "should": [
            { "match": { "title": "Elasticsearch" }},
            { "match": { "title": "Solr" }} 
          ],
          "must": { "match": { "authors": "clinton gormely" }} 
        }
      },
      "must_not": { "match": {"authors": "radu gheorge" }}
    }
  }
}
```

## More Fuzzy Queries
Fuzzy matching can be enabled on Match and Multi-Match queries to catch spelling errors. The degree of fuzziness is specified based on the  [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance)  from the original word, i.e. the number of one character changes that need to be made to one string to make it the same as another string.
```
POST /bookdb_index/book/_search
{
    "query": {
        "multi_match" : {
            "query" : "comprihensiv guide",
            "fields": ["title", "summary"],
            "fuzziness": "AUTO"
        }
    },
    "_source": ["title", "summary", "publish_date"],
    "size": 1
}
```

## Wildcard Query
Wildcard queries allow you to specify a pattern to match instead of the entire term. `?` matches any character and `*`matches zero or more characters. For example, to find all records that have an author whose name begins with the letter 't'
```
POST /bookdb_index/book/_search
{
    "query": {
        "wildcard" : {
            "authors" : "t*"
        }
    },
    "_source": ["title", "authors"],
    "highlight": {
        "fields" : {
            "authors" : {}
        }
    }
}
```

## Regexp Query
Regexp queries allow you to specify more complex patterns than wildcard queries.
```
POST /bookdb_index/book/_search
{
    "query": {
        "regexp" : {
            "authors" : "t[a-z]*y"
        }
    },
    "_source": ["title", "authors"],
    "highlight": {
        "fields" : {
            "authors" : {}
        }
    }
}
```

## Match Phrase Prefix
Match phrase prefix queries provide search-as-you-type or a poor man’s version of autocomplete at query time without needing to prepare your data in any way. Like the `match_phrase` query, it accepts a `slop` parameter to make the word order and relative positions somewhat less rigid. It also accepts the `max_expansions` parameter to limit the number of terms matched in order to reduce resource intensity.
```
POST /bookdb_index/book/_search
{
    "query": {
        "match_phrase_prefix" : {
            "summary": {
                "query": "search en",
                "slop": 3,
                "max_expansions": 10
            }
        }
    },
    "_source": [ "title", "summary", "publish_date" ]
}
```

## Query String
The `query_string` query provides a means of executing `multi_match` queries, bool queries, boosting, fuzzy matching, wildcards, regexp, and range queries in a concise shorthand syntax. In the following example, we execute a fuzzy search for the terms “search algorithm” in which one of the book authors is “grant ingersoll” or “tom morton.” We search all fields but apply a boost of 2 to the summary field.
```
POST /bookdb_index/book/_search
{
    "query": {
        "query_string" : {
            "query": "(saerch~1 algorithm~1) AND (grant ingersoll)  OR (tom morton)",
            "fields": ["title", "authors" , "summary^2"]
        }
    },
    "_source": [ "title", "summary", "authors" ],
    "highlight": {
        "fields" : {
            "summary" : {}
        }
    }
}
```

## Simple Query String
The `simple_query_string` query is a version of the `query_string` query that is more suitable for use in a single search box that is exposed to users because it replaces the use of `AND`/`OR`/`NOT` with `+`/`|`/`-`, respectively, and it discards invalid parts of a query instead of throwing an exception if a user makes a mistake.
```
POST /bookdb_index/book/_search
{
    "query": {
        "simple_query_string" : {
            "query": "(saerch~1 algorithm~1) + (grant ingersoll)  | (tom morton)",
            "fields": ["title", "authors" , "summary^2"]
        }
    },
    "_source": [ "title", "summary", "authors" ],
    "highlight": {
        "fields" : {
            "summary" : {}
        }
    }
}
```

## Term/Terms Query
The above examples have been examples of full-text search. Sometimes we are more interested in a structured search in which we want to find an exact match and return the results. The `term`and `terms` queries help us here. In the below example, we are searching for all books in our index published by Manning Publications.
```
POST /bookdb_index/book/_search
{
    "query": {
        "term" : {
            "publisher": "manning"
        }
    },
    "_source" : ["title","publish_date","publisher"]
}
```
Multiple terms can be specified by using the terms keyword instead and passing in an array of search terms.

## Term Query - Sorted
Term queries results (like any other query results) can easily be sorted. Multi-level sorting is also allowed.
```
POST /bookdb_index/book/_search
{
    "query": {
        "term" : {
            "publisher": "manning"
        }
    },
    "_source" : ["title","publish_date","publisher"],
    "sort": [
        { "publish_date": {"order":"desc"}}
    ]
}
```
**NOTE:** In ES6, to sort or aggregate by a text field, like a title, for example, you would need to enable fielddata on that field. 

## Range Query
Another structured query example is the range query. In this example, we search for books published in 2015.
```
POST /bookdb_index/book/_search
{
    "query": {
        "range" : {
            "publish_date": {
                "gte": "2015-01-01",
                "lte": "2015-12-31"
            }
        }
    },
    "_source" : ["title","publish_date","publisher"]
}
```

## Function Score: Field Value Factor
There may be a case where you want to factor in the value of a particular field in your document into the calculation of the relevance score. This is typical in scenarios where you want the boost the relevance of a document based on its popularity. In our example, we would like the more popular books (as judged by the number of reviews) to be boosted. This is possible using the `field_value_factor` function score.
```
POST /bookdb_index/book/_search
{
    "query": {
        "function_score": {
            "query": {
                "multi_match" : {
                    "query" : "search engine",
                    "fields": ["title", "summary"]
                }
            },
            "field_value_factor": {
                "field" : "num_reviews",
                "modifier": "log1p",
                "factor" : 2
            }
        }
    },
    "_source": ["title", "summary", "publish_date", "num_reviews"]
}
```

## Function Score: Decay Functions
Suppose that instead of wanting to boost incrementally by the value of a field, you have an ideal value you want to target and you want the boost factor to decay the further away you move from the value. This is typically useful in boosts based on lat/long, numeric fields like price, or dates. In our contrived example, we are searching for books on “search engines” ideally published around June 2014.
```
POST /bookdb_index/book/_search
{
    "query": {
        "function_score": {
            "query": {
                "multi_match" : {
                    "query" : "search engine",
                    "fields": ["title", "summary"]
                }
            },
            "functions": [
                {
                    "exp": {
                        "publish_date" : {
                            "origin": "2014-06-15",
                            "offset": "7d",
                            "scale" : "30d"
                        }
                    }
                }
            ],
            "boost_mode" : "replace"
        }
    },
    "_source": ["title", "summary", "publish_date", "num_reviews"]
}
```

## Function Score: Script Scoring
In the case where the built-in scoring functions do not meet your needs, there is the option to specify a Groovy script to use for scoring. In our example, we want to specify a script that takes into consideration the `publish_date` before deciding how much to factor in the number of reviews. Newer books may not have as many reviews yet so they should not be penalized for that.

The scoring script looks like this:
```
publish_date = doc['publish_date'].value
num_reviews = doc['num_reviews'].value

if (publish_date > Date.parse('yyyy-MM-dd', threshold).getTime()) {
  my_score = Math.log(2.5 + num_reviews)
} else {
  my_score = Math.log(1 + num_reviews)
}
return my_score
```

To use a scoring script dynamically, we use the `script_score` parameter
```
POST /bookdb_index/book/_search
{
    "query": {
        "function_score": {
            "query": {
                "multi_match" : {
                    "query" : "search engine",
                    "fields": ["title", "summary"]
                }
            },
            "functions": [
                {
                    "script_score": {
                        "params" : {
                            "threshold": "2015-07-30"
                        },
                        "script": "publish_date = doc['publish_date'].value; num_reviews = doc['num_reviews'].value; if (publish_date > Date.parse('yyyy-MM-dd', threshold).getTime()) { return log(2.5 + num_reviews) }; return log(1 + num_reviews);"

                    }
                }
            ]
        }
    },
    "_source": ["title", "summary", "publish_date", "num_reviews"]
}
```

## Lab complete