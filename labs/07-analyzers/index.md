# Elastic Stack Lab07
In this lab we will be playing around with different analyzers. These analyzers determine what type of search results we get when querying full text.  

There are two different types of analyzers for querying text fields. 
* Keyword
* Text

**Keyword**: 
The `keyword` type is much more rigid and is meant to search for specific full words, using case-sensitive matching.  This means that if I searched for `is` I wouldn’t get results for `this`  or `his`.  I also would not see text containing `Is`, `IS` or `iS`.

**Text**:
The `text` type is much more flexible and enables analyzers to search text fields however you want.  Partial matching, stemming, case-insensitive, case-sensitive, synonyms etc.. 

If I searched for `is` I would get back all words container `is`

Now let’s demonstrate this through a lab. 

## Text type search. 
We are going to do a search for “Star Trek” across our `movies` index and review the results. 

In a terminal use `curl` with a `GET` request to search the `movies` index with the following data 
```json
{
  "query": {
    "match": {
       "title": "Star Trek"
    }
  }
}
``` 

Now review the results, did you get what you were expecting? 
Pay attention to the first result, why is it different than what you searched for? 

If the index has a small dataset but a large number of shards this can lead to unexpected results.  For example we searched for “Star Trek” but because we are using the `text` mapping for `title` it actually looked in all the different shards for any documents containing the words `Star` and `Trek`.  The inverse document frequency is computed per shard and we had 2 matching documents with `Star` , one of the shards had higher relevancy score so `Star Wars` was returned first.  

In production you will have more documents and larger datasets so this won’t be an issue. 

Now let’s try another text search but this time we are going to use the query term `match_phrase` and see what the result is. 

In a terminal use `curl` with a `GET` request to search the `movies` index with the following data 
```json
{
  "query": {
    "match_phrase": {
       "genre": "sci"
    }
  }
}
``` 

What are the results? 
Default analyzer is being used for `genre` field, so it’s doing case-insensitive, full text search. 

## Start over 
Elasticsearch does not allow re-mapping of an index so if you decide later on you want to change mapping to get different search results you’ll need to delete the entire index and start over. 

Let’s go ahead and delete our `movies` index. 
```
curl -XDELETE 127.0.0.1:9200/movies
```


Now we need to create a new `movies` index with some changes to the way it searches text. 

```bash
curl -XPUT 127.0.0.1:9200/movies -d '
{
    "mappings" : {
        "movie": {
            "properties": {
                "id": {"type": "integer"},
                "year": {"type": "date"},
                "genre": {"type": "keyword"},
                "title": {"type": "text", "analyzer": "english"}
            }
        }
    }
}'
```

Notice we changed the `genre` to `keyword` so now it will only return exact matches,  and we won’t get any results if we do not type the entire keyword.  We also added a specific `analyzer` to the `title` field telling it to only search for matches in English.  This could lead to issues if we have foreign films in our index but it does illustrate how we can specify an analyzer for a field. 

Now that we’ve deleted everything and created the new mapping we need to repopulate our index. 

```bash
curl -XPUT 127.0.0.1:9200/movies/_bulk?pretty --data-binary @movies.json
```

The return should show that our index was populated with the contents of `movies.json`. 

## Search again 
Now that we’ve changed the mapping and re-populated our data let’s see if it responds as expected when we do a query for `sci` in the `genre` field. 

Using `curl` with the `GET` verb, query the Elasticsearch API `movies` index with the following data
```json
{ 
  "query": {
     "match": {
        "genre": "sci"
    }
  }
}
```

Did the `keyword` type work? Do you get any partial or case-insensitive results? 

Now let’s try to search for `sci-fi` and see what happens. 

Using `curl` with the `GET` verb, query the Elasticsearch API `movies` index with the following data
```json
{ 
  "query": {
     "match": {
        "genre": "sci-fi"
    }
  }
}
```

Are the results what you expected?
Why or why not? 

Now let’s try two query it one more time, paying attention to case.
```json
{ 
  "query": {
     "match": {
        "genre": "Sci-Fi"
    }
  }
}
```

Spend some time querying different genres and see what happens.  

# Lab Complete