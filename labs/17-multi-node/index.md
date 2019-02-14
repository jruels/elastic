# Elastic Stack Lab17

In this lab we are going to 
* Setup a second local Elasticsearch node 
* Observe how Elasticsearch automatically redistributes the shards to the new node 
* Stop original node and watch everything move to the other node 
* Restart orginal node and watch as Elasticsearch rebalances. 


Start by editing our `elasticsearch.yml` file
```
sudo vi /etc/elasticsearch/elasticsearch.yml
```
Set the following 
```
cluster.name: my-cluster
node.name: node-1
node.max_local_storage_nodes: 2
```
We then need to copy the `elasticsearch` directory 
```
sudo cp -rp /etc/elasticsearch /etc/elasticsearch-node2
```
In the new `elasticsearch-node2` directory edit the `elasticsearch.yml` file and set:
```
cluster.name: my-cluster
node.name: node-2
network.port: 9201
node.max_local_storage_nodes: 2
``` 

Now we've setup the configuration file for the new node in our cluster but there's still a little more. 
Elasticsearch requires a log directory so let's create a directory for our new node
```
sudo mkdir /var/log/elasticsearch-node2
```

Now we need to set correct permissons 
```
sudo chown elasticsearch.elasticsearch /var/log/elasticsearch-node2
```
Great, now we have one more small change to make. 
We need to copy the init script and update the `CONF` path.
To copy it run
```
sudo cp /usr/lib/systemd/system/elasticsearch.service /usr/lib/systemd/system/elasticsearch-node2.service
```
Now that we've copied it let's update the configuration.


In the file look for a line that has 
```
Environment=ES_PATH_CONF=/etc/elasticsearch
``` 

Update it so it points to your new directory
```
Environment=ES_PATH_CONF=/etc/elasticsearch-node2
```

Now we need to re-read the config
```

sudo /bin/systemctl daemon-reload
```

Let's stop Elasticsearch
```
sudo /bin/systemctl stop elasticsearch 
```

Now we'll start up the old and new version 
```
sudo /bin/systemctl start elasticsearch 
sudo /bin/systemctl start elasticsearch-node2
```

After a few minutes let's confirm the new node was added to the cluster 
```
curl -XGET 127.0.0.1:9200/_cluster/health?pretty
```

We should see the `number_of_nodes` has increased, due to adding another node.

Now if everything worked successfully `status` should go from yellow to green and you should see the shards rebalance. 

Now let's confirm it also shows the same from our new node by checking port `9201`
```
curl -XGET 127.0.0.1:9201/_cluster/health?pretty
```

We can now query either of them and get the same results. 

Now let's stop our original node and confirm that it fails over to thew new node we've added. 
```
sudo systemctl stop elasticsearch
```

Wait a couple minutes and check the cluster health
```
curl -XGET 127.0.0.1:9201/_cluster/health?pretty
```

Great! eveything failed over and it appears the end user wouldn't be affected at all. 

To confirm all the data is still available let's go ahead and query the new node for Shakespeare
```
curl -XGET 127.0.0.1:9201/shakespeare/_search?pretty
```

You should now see that the shards were automatically distributed between the old and the new node. 

Now if we start up our original node everything will rebalance and our cluster will go back to a green status 

Start up the service
```
sudo systemctl start elasticsearch 
```

Wait a few minutes and then query the cluster health 
```
curl -XGET 127.0.0.1:9200/_cluster/health?pretty
```

The above commands simulated a node failure and we can see that Elasticsearch handled it without any issues. 

# Lab Complete 
