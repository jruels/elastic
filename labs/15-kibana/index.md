# Elastic Stack Lab15

In this lab we will install and configure Kibana to use as a dashboard for Elasticsearch

Start by installing it. 
```
sudo apt-get update && sudo apt-get install -y kibana 
```

Now we need to edit the configuration file 
```
sudo vi /etc/kibana/kibana.yml
```

Uncomment and change the `server.host` to `0.0.0.0` to allow external connections. 

Now reload everything 
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
sudo /bin/systemctl start kibana.service
```

Now confirm Kibana was installed successfully by visiting 

http://127.0.0.1:5601 in your browser and you should see something like this. 


![](index/39FF03FE-F867-4B19-BF78-0C2844DB0018.png)

## Interact with Kibana 
Now that we have Kibana installed let's have some fun with it! 

First let's start by clicking the button in the top right that says "Set up index patterns". This will take you to a new screen where you can select the index you would like to see data for. 


![](index/1F4FC10F-7265-4030-9C57-C5AFF6A37217.png)

On this screen under `Index pattern` type in "shakespeare" and then click `Next step`, 

On the next screen click `Create index pattern`

Once that's complete you should see a screen like this

![](index/5528C188-7C40-4A3A-A796-A7625C314B9F.png)


Now let's go explore the index we just added. 

Click `Discover` in the top left, then search for 
`speaker:GLOUCESTER` and look at the results. 

You can see he was in a lot of plays.  Now  we can click on `play name` in the `Available Fields` section and see exactly what plays he was in.

For fun go ahead and click on the Visualize button 
This will take you to a screen that looks similar to this 
![](index/98873363-4E9A-41C2-861B-C0E72F15FB1A.png)


It is very easy to get dynamically generated graphs and charts using Kibana. 

Now click on Visualize on the left side bar.

Click on `Create a visualization`

Click Tag Cloud at the bottom of the screen and in the new window click `shakespeare`

![](index/3EEE69E5-4A45-409E-A50A-D4EF35DAB773.png)


One you do that click on `Tags` on the left under `Buckets`

![](index/E29AF805-A7FC-425C-9771-2154931756BB.png)


On this page fill in the following 
* Aggregation: Terms
* Field: text_entry
* Size: 100

After filling that in click the play button at the top of screen.

You should see a really interesting screen with the most popular text in Shakespeare's plays. 

![](index/52E2D94C-F9CE-4A42-9A91-3C77CEA57B4C.png)

Take some time to click around and explore Kibana. 

# Lab Complete 





