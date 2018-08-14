# Elastic Stack Lab16

In this lab we are going to configure Kibana to use the X-Pack plugin which will enable monitoring, alerting and many other features. 


Run the following to enable X-Pack in Kibana
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


