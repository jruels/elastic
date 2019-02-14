# Elastic Stack Lab16

In this lab we are going to install Filebeat and a couple plugins to play around with. 

Let's start by installing Filebeat
```
sudo apt-get update && sudo apt-get -y install filebeat
```

After the installation we need to install a couple plugins into Elasticsearch 

First enable geo-ip translation.
```
cd /usr/share/elasticsearch/
sudo bin/elasticsearch-plugin install ingest-geoip
```

Enable user-agent analytics
```
sudo bin/elasticsearch-plugin install ingest-user-agent
```

Now we need to restart Elasticsearch 
```
sudo /bin/systemctl stop elasticsearch.service
sudo /bin/systemctl start elasticsearch.service
```

It takes a few minutes for Elasticsearch to come back online so if you run the following commands and they report an error wait a bit and try again. 

Now let's go configure Filebeat.
```
cd /usr/share/filebeat/bin
sudo filebeat setup --dashboards
sudo mv /etc/filebeat/modules.d/apache2.yml.disabled /etc/filebeat/modules.d/apache2.yml
sudo vi /etc/filebeat/modules.d/apache2.yml  
```

In the file edit `var.paths`  for `access` logs to point to your home directory
`var.paths: ["/home/<user>/logs/access*"]`

Disable `error` logs by changing to look like below:
```
# Error logs 
  error:
    enabled: false
```

Now in our home directory we need to create `logs` directory and copy the access logs over
```
mkdir ~/logs 
cd ~/logs 
cp ~/access_log ~/logs/
```

We have an access log for Filebeat to work with so let's start it up

```
sudo /bin/systemctl start filebeat.service
```

## Kibana and Filebeats
Now that we have Filebeats shipping our logs to Elasticsearch we should be able to see those changes in Kibana. 

Start by browsing to the [Kibana dashboard](http://127.0.0.1:5601) 

Click on the "Management" tab to confirm a new index pattern for the `filebeat` logs was created. 

Great so now we've got all the data from Filebeat directly imported into Elasticsearch. 

Let's play around with this data now. 

In Kibana click on `Discover` on the left hand side, then. Where it says `shakespeare*` click the drop down arrow and choose `filebeat-*`

![](index/A0509C58-30A5-4BB4-B7F3-78962E5F3E38%204.png)

You're going to get "No results found".  Don't worry there is data it's just set to only show the last 15 minutes by default so we need to adjust the time range.

In the top right hand corner of the window click `Last 15 minutes` and change it to `Absolute` and then select the first week of May.


![](index/95356BFA-A130-4A1F-BC2A-DEADF8E2AFD8%204.png)

After this loads, you'll see a lot of log entries which can be filtered to provide valuable information. 

On the `Discover` screen, left hand side click on when hovering over `apache2.access.response_code:500`.  This will show you all the 500 errors for the selected time period. 

Now let's turn some of that info into graphs and charts. 

On the left click on `Dashboard` and choose  `[Filebeat Apache2] Access and error logs`


You should see something like this 
![](index/D9F9DE45-16DE-41E6-A501-E66DE926D044%204.png)

Now we can see how easy it is to stream logs to Elasticsearch and use Kibana to create graphs and charts. 

# Lab Complete 