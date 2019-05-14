# Elastic Stack Lab01
In this lab we will be installing and setting up Elasticsearch on an Ubuntu VM. 


## Install Elasticsearch 
Elasticsearch is based on Java, so we need to install a Java environment.

Install pre-requisites
```bash
sudo apt update
sudo apt install apt-transport-https
```


Install Java 
```
sudo apt install openjdk-8-jdk
```

Now we can install Elasticsearch itself.

First let’s add the Elasticsearch GPG key to our VM
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Now we need to add the Elasticsearch repo to our VM. 
```bash
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```

Finally let’s install Elasticsearch
```bash
sudo apt update 
sudo apt install elasticsearch
```

After this completes we need to allow external access to our Elasticsearch instance. 

Edit `elasticsearch.yml` 
```bash
sudo vi /etc/elasticsearch/elasticsearch.yml
```

Change `http.host` to `0.0.0.0`

![](index/0D7C537F-F1FA-4199-A63E-AA6EC3B74708%204.png)

 (in vi, use the arrow keys to move where you want to edit, then hit “i” to enter “insert mode” and make your edits. When done, hit `ESC` to exit “insert mode”, then type `:wq` to write your changes and quit vi.)

Now we have to restart the daemon so it re-reads the updated configuration file. 
```bash
sudo /bin/systemctl daemon-reload
```

Enable the `Elasticsearch` service and start it up.
```bash
sudo /bin/systemctl enable elasticsearch.service
sudo /bin/systemctl start elasticsearch.service
```

If everything restarted without any errors Elasticsearch has been successfully installed! 

Let’s confirm it is working as expected by connecting to the API.
```bash
curl 127.0.0.1:9200
```

```bash
$ curl 127.0.0.1:9200
{
  "name" : "itdyml7",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "lRpAt5psT1Cr-u6hS_bc2Q",
  "version" : {
    "number" : "6.2.3",
    "build_hash" : "c59ff00",
    "build_date" : "2018-03-13T10:06:29.741383Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

You can also test it by loading http://VMIP:9200 in a browser, and if you see something like the following it’s working correctly. 
![](index/05CDF398-09D6-4AE2-BA56-7A5BAA985A2D%208.png)

## Loading data into Elasticsearch 
Now that we have Elasticsearch installed it needs some data to aggregate and index.  Let’s go ahead and load in the complete works of William Shakespeare 

Download and create the mapping 
```bash
wget http://bit.ly/es-shakes-mapping -O shakes-mapping.json
curl -H 'Content-Type: application/json' -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json
```

Download the data 
```bash
wget http://bit.ly/es-shakes-data -O shakespeare_6.0.json
```

Now we are going to load this data into Elasticsearch through it’s API
```bash
curl -H 'Content-Type: application/json' -X POST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary  @shakespeare_6.0.json
```

And finally let’s go ahead and search the data we just inserted. 
```bash
curl -H 'Content-Type: application/json' -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
"query" : {
"match_phrase" : {
"text_entry" : "to be or not to be"
}
}
}
'
```

We are searching all of the data we inserted for “to be or not to be” and our result is…   Wow, pulled it out very quickly and we now know that it came from Hamlet.
```json
{
  "took" : 153,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 13.874454,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_type" : "doc",
        "_id" : "34229",
        "_score" : 13.874454,
        "_source" : {
          "type" : "line",
          "line_id" : 34230,
          "play_name" : "Hamlet",
          "speech_number" : 19,
          "line_number" : "3.1.64",
          "speaker" : "HAMLET",
          "text_entry" : "To be, or not to be: that is the question:"
        }
      }
    ]
  }
}

```

## Lab Complete