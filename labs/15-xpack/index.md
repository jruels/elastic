# Elastic Stack Lab15

In this lab we are going to install the X-Pack plugin which will enable monitoring and alerting in Elasticsearch. 

Start by running the following to install it.
```
cd /usr/share/elasticsearch
sudo bin/elasticsearch-plugin install x-pack 
```

Hit 'Y' to confirm each of the questions. 

Now that' it's installed let's go ahead and configure it. 
Edit `/etc/elasticsearch/elasticsearch.yml`  to disable security on it so we can use it for free rather than paying a subscription fee. 

Add the following to `/etc/elasticsearch/elasticsearch.yml` 
```
xpack.security.enabled: false
```

Now we need to restart Elasticsearch. 

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl stop elasticsearch.service
sudo /bin/systemctl start elasticsearch.service
````

Now let's setup Kibana to work with X-Pack 
```
cd /usr/share/kibana
sudo -u kibana bin/kibana-plugin install x-pack
sudo /bin/systemctl stop kibana.service
sudo /bin/systemctl start kibana.service
```
 
Open up Kibana in a browser: 
http://127.0.0.1:5601

You may be prompted for a username/password, if you are use the following
* username: elastic
* password: changed

Once in Kibana, click `Monitoring`on the left side. 

You will see that it says "No monitoring data is available'...  don't worry that's not actually the case.  We just need to adjust the date we are looking at. 

In the top right corner click on the timeframe and change it to "Last 5 years", now you should have some monitoring recommendations.

Now click around and see what data is being monitored. 

# Lab Complete


