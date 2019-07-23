# Elastic Stack Lab04
Before beginning this lab create the alias for `curl` discussed in the slides. 

Open `~/.bashrc` in vim or your favorite editor and add: 
```
alias curl="/usr/bin/curl -H 'Content-type: application/json' "
```
Now that source the updated `~/.bashrc` file to apply alias. 
```
source ~/.bashrc
```

In this lab you will be importing data from the MovieLens dataset we downloaded earlier.  

## Import Single Document
Now that we have the mapping configured for ‘year’ we can use `curl` to insert a specific movie. 

Remember that for the purposes of this class we are using `curl` but in most other environments you would interact with the Elasticsearch API through a programming language, and there are client libraries available for `Python`, `Java`, `Golang` etc.. 

### Insert movie
```bash
curl -XPUT 127.0.0.1:9200/movies/movie/109488 -d '
{
"genre" : ["IMAX","Sci-Fi"], "title" : "Interstellar", "year" :2014
}'
```

Let’s break down this command and go through each section. 
We start with the `curl` command which sends our `PUT` request to Elasticsearch’s API using a `REST` call. 

The data section contains `genre` which is actually a list because we want to map it to `IMAP` and `Sci-Fi` genres. We then give it a `title` and provide the `year` it was released. 

If this ran successfully you should see something like the following confirming it was added.

```json
{"_index":"movies","_type":"movie","_id":"109488","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
```

### Retrieve movie
We can now run a `curl` command with the `GET` verb to confirm the movie is actually in Elasticsearch.  We are going to pass `_search` without any other values so that it will show us all documents inside of our `movies` index. 

```bash
curl -XGET 127.0.0.1:9200/movies/movie/_search?pretty
```

You should now see confirmation that the movie was added. 

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "109488",
        "_score" : 1.0,
        "_source" : {
          "genre" : [
            "IMAP",
            "Sci-Fi"
          ],
          "title" : "Interstellar",
          "year" : 2014
        }
      }
    ]
  }
}
```

Awesome! You’ve successfully inserted your first document in Elasticsearch! 

### Congrats! 

## Import Many Documents
Elasticsearch has a bulk import endpoint which does exactly what you would think… it allows you to import a bunch of documents at the same time. 

You will be downloading a `JSON` file that contains a few movies and using `curl` to import that data into Elasticsearch. 

First let’s start by pulling down the data file. 
```bash
wget http://bit.ly/es-movies-data -O movies.json
```

Now let’s look inside and confirm the data looks like we expect. 
```bash
vi movies.json 
```

You should now see the same list of movies from the slide. 
```json
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "135569" } }
{ "id": "135569", "title" : "Star Trek Beyond", "year":2016 , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "122886" } }
{ "id": "122886", "title" : "Star Wars: Episode VII - The Force Awakens", "year":2015 , "genre":["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "109487" } }
{ "id": "109487", "title" : "Interstellar", "year":2014 , "genre":["Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "58559" } }
{ "id": "58559", "title" : "Dark Knight, The", "year":2008 , "genre":["Action", "Crime", "Drama", "IMAX"] }
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "1924" } }
{ "id": "1924", "title" : "Plan 9 from Outer Space", "year":1959 , "genre":["Horror", "Sci-Fi"] }


```

Let’s break this down a little.  

Each movie has a pair of lines. The first line is the create command, it creates the index for the document we will be uploading. This includes the `index`, `type` and `id`.  This information is used to hash the document to a specific shard and send the data off to the matching shard. 

After that the individual documents are imported into the index with the fields needed to query them. 

We are now going to insert all 5 movies at once. 
```bash
curl -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @movies.json
```

After running that command you should get something back stating whether all the movies were inserted successfully.  Take a look at that and see if any of the imports failed.

Why or why not? 

```json
{
  "took" : 42,
  "errors" : false,
  "items" : [
    {
      "create" : {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "135569",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
..<snip>
```

Now that we’ve imported these documents let’s see what we can do with them. 

List all documents in the `movies` index
```bash
curl -XGET 127.0.0.1:9200/movies/_search?pretty
```


Search for all movies with the word `the` in the title. 
```bash
curl -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
    "query": {
        "query_string" : {
            "default_field" : "title",
            "query" : "the"
        }
    }
}
'
```

Now play around and search using other fields. 

Can you find all the movies that are in the `Action` genre?

What other fields can you query? 

# Lab Complete
