# Elastic Stack Lab09
In this lab you are going to play around with the “URI Search” format against your Elasticsearch API, then use the correct `JSON` approach and finally use phrase matching and slop wildcards.

Remember this is something that is usual for one-off quick checks, not for production. 

Start out by using the query from the slides. 
```bash
curl -XGET "127.0.0.1:9200/movies/movie/_search?q=title:star&pretty"
```

You should get back all movies with star in the title. 

Now let’s try a more advanced query and see if we can reach the limits of URI Search without `url-encoded` data. 

Give it a shot 
```bash
curl -XGET "127.0.0.1:9200/movies/movie/_search?q=+year:>2010+title:trek&pretty"
```

That query should return any movies that have been released since 2010 and contain the word “star” in the title. 

Did that return what we were requesting? 

Nope, the query we tried to send was not encoded so curl couldn't interpret it correctly. This is why “URI Search” can be difficult and return unexpected results. 

Now let’s go ahead and run that again but we’ll URL encode it. 
```bash
curl -XGET "127.0.0.1:9200/movies/movie/_search?q=%2Byear%3A%3E2010+%2Btitle%3Atrek&pretty"
```

As you can see this is tedious and confusing, not really as simple as it seems at first. 

## JSON search 
Now we’re going to do the same searches we did previously using “URI Search” but instead we’re going to do things “the right way” by using a full JSON body for our searches. 

Start by querying for any movies with the word `star` in the title. 
```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match": {
            "title": "star"
        }
    }
}'
```

As you can see this is much easier to read and there’s no URI encoding required. 

Now we are going to use a most complex search. This query will contain a boolean described by a `must` clause, and term query where the title must be `trek`. We are also passing a range filter to test and make sure the `year` field is greater than or equal to 2010.  

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "bool": {
            "must": {"term": {"title": "trek"}},
            "filter": {"range" : {"year": {"gte": 2010}}}
                }
        }
}'
```

It worked! We got the only movie in our database that contained the word “trek” in the title and was released after the year 2010. 


## Phrase search
Sometimes you don’t want to search for single words or partial matches.  You want to get results for an entire phrase.  Elasticsearch has a great way of accomplishing this, the `match_phrase` query term.  Using this we can provide a full phrase and get the results we’re looking for. 

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
    "query":{
        "match_phrase": {
            "title": "star wars"
        }
    }
}'
```

## Slop search 
If you are not quite sure what is contained in a phrase but you know a few words you can use `slop`. This allows you to specify how many words are in between the words you’re searching for. 

Let’s do a `match` query for `Star Wars` without using `match_phrase` and review the results. 

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
    "query":{
        "match": {
            "title": "star wars"
        }
    }
}'
```

As you can see the results contain `Star Wars` which is what we were searching for, but they also returned anything containing `Star` or `Wars` with a higher relevance for `Star Wars`. If you only want `Star Wars` we can use a `match_phrase` query. 

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
    "query":{
        "match_phrase": {
            "title": "star wars"
        }
    }
}'
```

Now the results are only movies containing `Star Wars`

Now let’s add `slop` and see what that does. 
```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
    "query":{
        "match_phrase": {
            "title": {"query": "star beyond", "slop": 1}
        }
    }
}'
```

The results now show us any movies with `Star` and `Beyond` in that order, but because of `slop` we can have 1 word in between, or on the left or right and it will also return results for reverse order such as `Beyond Star`

Now let’s look at a proximity query.  Use the same as the previous query but specify a higher slop value, in this case let’s set it to 100 and see what the results are. 
```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty -d'
{
    "query":{
        "match_phrase": {
            "title": {"query": "star beyond", "slop": 100}
        }
    }
}'
```

That wasn’t very exciting, our dataset is too small to really see any difference but if we did have a larger dataset it would return all results that have `Star` and `beyond` with up to 100 words in between and sort them by relevance. 


# Lab Complete 

