# Elastic Stack Lab05
In this lab you will be updating documents in the `movies` index with new versions. 

## Update Interstellar 
We are going to update the title of the movie `Interstellar` by using the `curl` command.  

First let’s see what data is already inserted for this movie. 

```bash
curl -XGET 127.0.0.1:9200/movies/movie/109487?pretty
```

This will output: 

```json
{
  "_index" : "movies",
  "_type" : "movie",
  "_id" : "109487",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "id" : "109487",
    "title" : "Interstellar",
    "year" : 2014,
    "genre" : [
      "Sci-Fi",
      "IMAX"
    ]
  }
}
```

Let’s pay careful attention to each of the fields and particularly the `version` field. Notice it is set to `1` which means this document has never been updated or changed. 

Now that we know what it currently looks like let’s go ahead and update the title

```bash
curl -XPOST 127.0.0.1:9200/movies/movie/109487/_update -d '
{
    "doc": {
        "title": "Outerstellar"
    }
}'
```

This command connects to the Elasticsearch API, and updates the movie with id `109487` to have a new title. To confirm this change was made successfully we can run

```bash
curl -XGET 127.0.0.1:9200/movies/movie/109487?pretty
```

You should see something like: 
```json
{
  "_index" : "movies",
  "_type" : "movie",
  "_id" : "109487",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "id" : "109487",
    "title" : "Outerstellar",
    "year" : 2014,
    "genre" : [
      "Sci-Fi",
      "IMAX"
    ]
  }
}
```

As you can see the title was updated and the `_version` field incremented.  

Ok well that title isn’t as clever as I originally thought.  Let’s go ahead and try to create a new document with the correct title. 

Using `curl`  with the `POST` verb create a new document with the same `_id` of `109487` and the following data 
```json
{
    "genre" : ["IMAP","Sci-FI"], 
    "title" : "Interstellar", 
    "year" :2014
}
```

What happened?  Does the document still have version `2`

In the output you’ll notice that the `_version` incremented to `3`  even though we told Elasticsearch to create a new document with id `109487` it is smart enough to realize it already exists and to just update it.  Check the `result` field and see what it says. 

Now play around and update a few more documents. 

# Lab Complete 
