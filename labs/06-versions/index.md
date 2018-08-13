# Elastic Stack Lab06
In this lab you will be learning more about the `_version` field and how it can help with concurrency in Elasticsearch’s distributed architecture. 

## Check version number of a document 
* Use `curl` with the `GET` verb to get a list of the documents in the `movies` index
* Choose one of these movies and note the `id` , and `_version` fields. 

Elasticsearch allows us to specify a specific version when we go to update a document, this is so we can use optimal concurrency control to ensure the correct version is updated and there’s no conflicts.

## Update specific version of document
To update a specific version we run a command like the following, but remember to change `<movieid`, `version`, `genres`, `title`, and `year` to match the movie you chose! 
```bash
curl -XPUT 127.0.0.1:9200/movies/movie/<movieid>?version=<movie_version> -d '
{
  "genres" : ["IMAX", "Sci-Fi"],
  "title" : "Interspeller",
  "year" : 2014
}'
```

After applying this update confirm it was successful by executing a `GET` against the Elasticsearch API. 

If it was successful continue on otherwise check to make sure you don’t have any syntax errors, and if needed get the instructor. 

Now that we’ve successfully updated our movie document let’s try to update the same version again and see what happens. 

Run the exact same command as above and look at the output. 

## Retry on conflict 
Now we are going to send an updating using `POST` and this time we’re going to add the option to retry if we are updating an older version. 

I will be using the following because the movie I chose was `Interstellar` but you should replace this with the info for the movie you chose in the previous step. 
```bash
curl -XPOST 127.0.0.1:9200/movies/movie/109487/_update?retry_on_conflict=5 -d '
{
  "doc": {
    "title" : "Interstellar"
  }
}'
```

Now confirm it was successful, but running `curl` with `GET` against the movie `id` you updated. 

Spend some time playing around with version conflicts and `retry_on_conflict` updates.

# Lab Complete 