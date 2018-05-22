# Elastic Stack Lab12

In this lab we are going to look at using client libraries to import data and interact with the Elasticsearch API.

## Python - import data
Start by making sure the `unzip` utility is installed so we can unzip the movies file. 

```bash
sudo apt-get install -y unzip 
```

Now let's download the MovieLens data. 
```bash
wget http://files.grouplens.org/datasets/movielens/ml-latest-small.zip
```

After downloading we just need to unzip 
```bash
unzip ml-latest-small.zip 
```

Now let's download the `Python` script we'll be using for manipulating and importing the movie data. 
```bash
wget http://bit.ly/es-python-import -O MoviesToJson.py
```

The `Python` script opens up the movies CSV file, loads the data into a dictionary and then writes out a new output file with the fields needed for importing. 

Now let's run the script to convert the `movies.csv` to a `JSON` format. 
```bash
python3 MoviesToJson.py > moremovies.json
```

Review `moremovies.json` and confirm it looks correct. 

You should see a format similar to: 
```json
{ "create" : { "_index": "movies", "_type": "movie", "_id" : "1" } }
{ "id": "1", "title": "Toy Story", "year":1995, "genre":["Adventure","Animation","Children","Comedy","Fantasy"] }
{ "create" : { "_index": "movies", "_type": "movie", "_id" : "2" } }
{ "id": "2", "title": "Jumanji", "year":1995, "genre":["Adventure","Children","Fantasy"] }
```

While this dataset is considered "small" it still contains over 18,000 lines!   This is a lot more data to play around with than we've had to this point. 

Now we need to delete our `movies` index so we can reimport using our new dataset. 

```bash
curl -XDELETE 127.0.0.1:9200/movies
```

After deleting the index let's go ahead and recreate it with the new data.

```bash
curl -XPUT 127.0.0.1:9200/_bulk --data-binary @moremovies.json
```

Now that everything was imported successfully let's do a search and see what is returned.  In this example we are going to use the "URI Search". 

```bash
curl -XGET "127.0.0.1:9200/movies/_search?q=title:ice%20&pretty"
```

Alright, so we got back a ton of results!  Now that we have a lot more data to play around with we should start to see better relevancy results.

Now let's go ahead and use a client side library to interact with the API instead of  `curl`

## Python - ratings 
To run our `Python` script we must first install `pip` and some modules from it. 

```bash
sudo apt install -y python3-pip
```

Now that `pip` is installed let's use it to install the `Python` Elasticsearch module.
```bash
sudo pip3 install elasticsearch 
```

After that's complete we need to download the script. 
```bash
wget http://bit.ly/es-py-ratings -O IndexRatings.py
```

Now that we've downloaded that it's time to run it! 

```bash
python3 IndexRatings.py
```

Ok, now let's use our trusty `curl` command to confirm everything was created successfully. 

```bash
curl -XGET 127.0.0.1:9200/ratings/_search?pretty
```

And if everything went smoothly we should see a list of movies with their ratings.

We can import many other files as well with simple edits to our `Python` script. 

## Python - tags 
Copy the `IndexRatings.py` script to `IndexTags.py` and modify it to import all of the data from the `tags.csv` file.

```bash
python3 IndexTags.py
```

After that completes confirm the data was imported as expected. 

```bash
curl -XGET 127.0.0.1:9200/tags/_search?pretty
```

Confirm the tags were imported successfully. 

# Lab Complete 