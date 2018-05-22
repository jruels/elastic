# Elastic Stack Lab 14

In this lab we are going to use aggregates with our indices. 

Let's start by running a query on the `ratings` index and requesting ratings for all movies.

```bash
curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d '
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

In the above command we specified `size=0` so that we avoid having Elasticsearch return all documents that match, and only returning the results of our ratings query.

You can now see the number of documents that have specific ratings.  Not surprising 4.0 is the largest rating used. 

Now let's try a query that's a bit more complicated.  We are going to pull back only 5.0 rated movies. 

```bash
curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d '
{
    "query": {
        "match": { "rating": 5.0 }
    },
    "aggs": {
        "ratings": {
            "terms": { "field": "rating" }
        }
    }
}'
             
```

A good practice for using aggregate is to start with a query block to narrow down the results and then use an aggregation to add up the remaining documents. 

Now let's get an aggregation for the average rating of "Star Wars Episode IV"

```bash
curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d '
{
    "query": {
        "match_phrase" : {"title" : "Star Wars Episode IV" }
        }
        ,
        "aggs": {
                "avg_rating": {
                     "avg": {"field" : "rating"}
        }
    }
}'
```

## Histograms 
Histograms are a great way to retrieve data that can easily be used for visual graphs. 

Let's start by creating a query which will show us ratings grouped by whole number. 
```bash
curl -XGET '127.0.0.1:9200/ratings/rating/_search?size=0&pretty' -d '
{
    "aggs": {
        "whole_ratings": {
            "histogram": {
                "field": "rating",
                 "interval": 1.0
            }
        }
    }
}'
```

The output of this query should show you ratings grouped by whole number. 

Now let's bucket up all the movies by release year. 
```bash
curl -XGET '127.0.0.1:9200/movies/movie/_search?size=0&pretty' -d '
{
    "aggs": {
        "release": {
            "histogram": {
               "field": "year",
               "interval": 10
            }
        }
    }
}'
```

This query will return results of how many movies were released each decade. 

# Lab Complete
