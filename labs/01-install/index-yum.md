# Elastic Stack Lab01
In this lab we will be installing and setting up Elasticsearch on a CentOS/RHEL VM. 

These directions are based on the official documentation available at:

https://www.elastic.co/guide/en/elasticsearch/reference/7.6/rpm.html#rpm-repo

## Install Elasticsearch 
Elasticsearch is based on Java, so we need to install a Java environment.

Install pre-requisites
```bash
sudo yum update -y
```

Check for a Java 8 VM
```bash
[student@ip-172-30-0-35 ~]$ java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (build 25.242-b08, mixed mode)
```

Install Java if not installed 
```bash
sudo yum install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64
```

Now we can install Elasticsearch itself.

Now we need to add the Elasticsearch repo to our VM. 
```bash
cd ~/downloads
wget https://github.com/calvinvette/elastic/blob/master/elasticsearch6.repo
wget https://github.com/calvinvette/elastic/blob/master/elasticsearch7.repo
sudo cp downloads/elasticsearch* /etc/yum.repos.d/
```

Finally let’s install Elasticsearch (note that the repos are disabled by default):
```bash
sudo yum update -y
# For ES6, use: 
sudo yum install --enablerepo=elasticsearch6 elasticsearch
# For ES7, use: 
sudo yum install --enablerepo=elasticsearch7 elasticsearch
```

If the yum install fails (as of this writing, the repo is returning 404-Not Found), you can download the RPMs 
directly and install them:

```bash
# For ElasticSearch 7
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-x86_64.rpm
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-x86_64.rpm.sha512
shasum -a 512 -c elasticsearch-7.6.2-x86_64.rpm.sha512 
sudo rpm --install elasticsearch-7.6.2-x86_64.rpm
```

```bash
# For ElasticSearch 6
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.8.rpm
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.8.rpm.sha512
shasum -a 512 -c elasticsearch-6.8.8.rpm.sha512 
sudo rpm --install elasticsearch-6.8.8.rpm
```


After this completes we need to allow external access to our Elasticsearch instance. 

Edit `elasticsearch.yml` 
```bash
sudo vi /etc/elasticsearch/elasticsearch.yml
```

Change `http.host` (earlier ES6) or `network.host` (later ES6 and ES7) around line 55 to `0.0.0.0`

![](index/0D7C537F-F1FA-4199-A63E-AA6EC3B74708%204.png)

 (in vi, use the arrow keys to move where you want to edit, then hit “i” to enter “insert mode” and make your edits. When done, hit `ESC` to exit “insert mode”, then type `:wq` to write your changes and quit vi.)

Modify the init script to export a proper JAVA_HOME, around line 59:
```bash
sudo vi /etc/init.d/elasticsearch
```

Change this:

```
export JAVA_HOME
```

to this (or an appropriate JAVA_HOME):

```
export JAVA_HOME=/usr/lib/jvm/java
```

Enable the `Elasticsearch` service and start it up.
```bash
sudo chkconfig --add elasticsearch
sudo service elasticsearch start
```

If everything restarted without any errors Elasticsearch has been successfully installed! 

Let’s confirm it is working as expected by connecting to the API.
```bash
curl 127.0.0.1:9200
```

```bash
$ curl 127.0.0.1:9200
{
  "name" : "QDJyUOw",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "FS96-DAATMKss1b-0xTL7g",
  "version" : {
    "number" : "6.8.8",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "2f4c224",
    "build_date" : "2020-03-18T23:22:18.622755Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.2",
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
