# Elastic Stack Lab10

In this lab we’re going to play around with pagination, sorting and more advanced filtering.  We’re also going to learn how to use `fuzziness` to match against mistyped searches. 

## Pagination 
If there are a lot of results it’s very helpful to use `pagination` so that we can return a small number of results. 

We’ll start out by using the URI Search to return the first 2 results in the `movies` index, no sorting, no relevancy, just pull down the first 2.  

```bash
curl -XGET '127.0.0.1:9200/movies/movie/_search?size=2&pretty'
```
Did your search only return 2 results? Is this what you expected? 

You may have noticed there is no `from` specified.  This is because by default it starts at 0, so since we want to see the first 2 results there’s no need to specify a `from` value. 


If someone was displaying this on a website and I wanted the next 2 results I would specify a `from` and set it to `2` since we just showed them `0` and `1`, the next would be `2`. 

Go ahead and run the above command but add `from=2` to the query. 

Did you get back the next 2? 

You’ve queried the Elasticsearch API using URI Search now let’s use a `JSON` body to clean this up a bit and show how it can be done. 

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
   "from": 2,
   "size": 2,
   "query": {"match": {"genre": "Sci-Fi"}}
}'
```

Your query should return 2 results from the 2nd page (if we were displaying on website).   Go ahead and look at the results that would have been displayed on the first page, and query for the results for the 3rd page. 

## Sorting 
Sorting is very easy! We can do it using URI Search or through a JSON body.  Remember that to sort a field it cannot use an analyzer and must be `keyword` type. 

Let’s sort all of the documents in the `movies` index by `year`
```bash
curl -XGET '127.0.0.1:9200/movies/movie/_search?sort=year&pretty'
```

We can see that it did sort the results by year as expected!  

Let’s try it again but now we’re going to sort by the `title` field.

```bash
curl -XGET '127.0.0.1:9200/movies/movie/_search?sort=title&pretty'
``` 

What happened? 

This error message is due to the fact the `title` field is analyzed and you can’t sort on an analyzed field, because it’s not actually sorting the contents of the field directly it’s storing the individual terms within the inverted index. 

Now we get to delete everything and create a new mapping and re-import the data to create a sub field of `keyword` type.

```bash
curl -XDELETE 127.0.0.1:9200/movies 
```

```bash
curl -XPUT 127.0.0.1:9200/movies -d '
{
    "mappings" : {
        "movie" : {
            "properties" : {
                "title" : {
                    "type" : "text",
                    "fields" : { "raw": {"type": "keyword"}}
                }
            }
        }
    }
}'
```
Great you just created a title field of type `text` which is analyzed and also a title.raw field which is of type `keyword`

Now use steps from previous labs to reimport the `movies.json` data.

After importing let’s try that sort query again using `title.raw` instead of `title` field. 

```bash
curl -XGET '127.0.0.1:9200/movies/movie/_search?sort=title.raw&pretty'
``` 

Are the movies sorted by title? Alphabetically? 

A subfield of type `keyword` allows you to sort any field. 

## Complex filter 
Filters are used for almost everything within Elasticsearch.  If you want to retrieve relevant data you will need to filter it. 

Here is an example of a complex filter that will return all `Sci-Fi` movies released between years `2010` and `2015` and do not have `trek` in the title. 
```bash
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

Now looking at this query go ahead and write one of your own that meets the following requirements. 
* Genre “Action” 
* Must NOT contain “Star” in the title 
* Released between years 2001 - 2010 

## Fuzziness Queries. 
Let’s start with a non-fuzzy query and see what happens when we misspell something simple.  

Start out by using a `match` query to search for `intersteller`  spelled with an `e` instead of an `a`

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match": {
            "title": "intersteller"
        }
    }
}'
```

What were the results? 

Since we didn’t get any results using a regular match query let’s see if we can use `fuzzy` to help us out. 

We are going to run a `fuzzy` querying so that it is more tolerant of mistakes. 
We need to specify the `value` we are searching for, and the `fuzziness` we are willing to tolerate. 

Let’s use the same query as above with a couple changes. 
```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "fuzzy": {
            "title": {"value": "intersteller", "fuzziness":1 }
        }
    }
}'
```

Alright so now that we’ve used a `fuzzy` query we are able to handle typos and mistakes of up to 1 character. 

Alright, let’s play around with fuzzy a little bit. Try and figure out the following scenarios on your own and if you need assistance speak with the instructor. 

* Perform a fuzzy query against any movie in the database with 2 different additional characters added.  Make sure that you adjust so that matches are returned. 

# Lab Complete 