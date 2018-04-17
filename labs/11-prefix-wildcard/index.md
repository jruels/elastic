# Elastic Stack Lab11

This lab is going to focus on prefix and wildcard queries, which both make retrieving the correct results much easier and you'll learn about predictive searching.  

We need to create a new mapping and to do that we have to delete the index. 

* Delete `movies` index using same steps as previous labs.

Now that’s complete let’s go ahead and create a new mapping for the `year` field as a `keyword` .

```bash
curl -XPUT 127.0.0.1:9200/movies/ -d '
{
    "mappings": {
        "movie":  {
            "properties": {
                "year": {
                    "type": "keyword"
                }
            }
        }
    }
}'
```

After creating this we can now sort and filter based on results for field `year`. Remember we will not be able to use analyzers on the `year` field any more because it is now a `keyword` type.

* Using steps from previous labs import the `movies.json` data into the `movies` index. 

Now we have a populated index with a field `year` that is type `keyword` and allows us to do some fun filtering of the results. 

To test this out let’s go ahead and run the following 
```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{ 
    "query": {
        "prefix": {
            "year": "201"
           }
    }
}'
```

You can see that it returns movies that were released any time between 2011 and 2019. 

Another option is to use a wildcard `*` in our searches.  

Let's give that a shot.
```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{ 
    "query": {
        "wildcard": {
            "year": "19*9"
           }
    }
}'
```

What result did you get? 

* Play around with prefix and wildcard searches. 

## Autocomplete 
We are now going to setup autocompletion of queries. This allows us to return suggested results immediately to our users as they are typing their queries. 

To begin we must delete our `movies` index so we can create a new mapping. At this point you should know how to do this without the full command being provided, for a refresher look at previous labs. 

* Delete `movies` index 

Create the following mapping
```bash
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
                        "type": "custom",  "tokenizer": "standard",  
                        "filter": ["lowercase",  
                        "autocomplete_filter"
                    ]
                }
            }
        }
    }
}'
```

As was explained in the slides this mapping will create an `autocomplete` filter which matches edge characters starting at 1 character and all the way up to 20 characters. It also creates a custom analyzer named `autocomplete` which uses the standard `lowercase` filter as well as our `autocomplete` filter. 

Now let's test out this new analyzer and see what it does when simple text is passed to it. 

```bash
curl -XGET 127.0.0.1:9200/movies/_analyze?pretty -d '
{
    "analyzer": "autocomplete",
    "text": "Sta"
}'
```

Looking through the results should show you an example of how the analyzer will handle queries. Review the results and determine if they are as expected. 


Now we need to apply this new analyzer to our `title` field and create a new mapping. 
```bash
curl -XPUT '127.0.0.1:9200/movies/_mapping/movie?pretty' -d '
{ 
    "movie": {
        "properties": {
            "title": { "type": "text", "analyzer": "autocomplete" }
        }
    }
}'
```

After creating the new mapping we need to re-populate the `movies` index with data. 

```bash
curl -XPUT 127.0.0.1:9200/movies/_bulk?pretty --data-binary @movies.json
```

Once we've repopulated the data let's go ahead and run a couple queries using the new mapping we just created. 

Start by running a simple 	`match` query for the `title` field searching for `sta`.

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match": {
            "title": "sta"
        }
    }
}'
```

What happened?  Did it return the results we were expecting? 

Well, if you remember we discussed how on the index side we want to use the n-grams filter, but on the query side we do not want to search for 's' 't' and 'a', we want to use the `standard` analyzer from the query side.

Let's update the above query to use a different analyzer. 
```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match": {
            "title": {"query": "sta", "analyzer": "standard"}
        }
    }
}'
```

Now when you run the above command what does it return? 

Are these results a little better? 

One thing to point out is that even though we are searching for  `Star Wars` our standard analyzer still matches all occurrences of `star` and `wars` , which is why results include "Star Trek" movies as well. 

The only way to make sure we get perfect results is to use completion suggesters, but they take a lot of engineering time and resources. 

# Lab Complete 