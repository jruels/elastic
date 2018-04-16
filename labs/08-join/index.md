# Elastic Stack Lab08
In this lab you will be creating a new index, new mappings, and relationships between the data in our Elasticsearch cluster. 

## Create new mapping 
Sometimes you have data that is related. In this example we are going to create a parent/child relationship between films and the franchise they are part of. 

```
curl -XPUT 127.0.0.1:9200/series -d '
{
    "mappings" : {
        "movie": {
            "properties": {
                "film_to_franchise": {"type": "join", "relations": {"franchise" : "film"}}
            }
        }
    }
}'
```

Notice we created new properties with a field `film_to_franchise` and a new `type` of `join` then we are saying franchise and film values are related. 

## Query new series data
Now that we have created the new index and mapping we need to populate it with data. 

Download the file containing the series information. 
```
wget http://bit.ly/es-series-data -O series.json
```

We review the contents of this file in class, but if you want to look through it again feel free. 

Now we are going to import this new data into our `series` index. 

```
curl -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @series.json
```

Confirm there were no errors in the output. 

Let’s run a query now to find all of the films that are part of the `Star Wars` franchise! 

To do this we are going to use the `has_parent` option with a `parent_type` of `franchise`. This should return all of the documents in our `series` index that are children of the `franchise`. 

```
curl -XGET 127.0.0.1:9200/series/movie/_search?pretty -d '
{
    "query" : {
        "has_parent": {"parent_type": "franchise", "query": { "match" : {"title": "Star Wars"}}}
    }
}'
```

What did you get back? 

Now let’s try to find the parent for a specific child. 
```
curl -XGET 127.0.0.1:9200/series/movie/_search?pretty -d '
{
    "query" : {
        "has_child": {
            "type": "film", "query": {"match": {"title": "The Force Awakens"}}
        }
    }
}'
```

Did it return the correct results? 

Remember that indexing parent/child joins can take a lot of performance, also need to make sure you index them specifically to the same shard. So this can be a complicated, expensive design and you should only do it if absolutely necessary. 

## Extra credit 
	* Import any other movie, that isn’t Star Wars, into the `series` index 
	* Perform a search to show all movies in the `series` index. 
	* Query for all movies in the “Star Wars” franchise and confirm your movie does not show up. 

# Lab Complete