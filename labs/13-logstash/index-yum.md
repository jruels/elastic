# Elastic Stack Lab13
In this lab we are going to install and configure Logstash. 

Run the following to install Logstash.  
```
sudo apt-get update
sudo yum install -y logstash 
```

If the Elastic repo is returning 404, use the RPMs directly:

ES7
```
wget https://artifacts.elastic.co/downloads/logstash/logstash-7.6.2.rpm
rpm install logstash-7.6.2.rpm
```

```
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.8.8.rpm
rpm install logstash-6.8.8.rpm
```

After it's installed we have to configure it.

Now create  `/etc/logstash/conf.d/logstash.conf` in vi 
And insert the following

```
input {
  file {
    path => "/home/student/access_log"
    start_position => "beginning"
  }
}

filter {
  if [path] =~ "access" {
    mutate { replace => { "type" => "apache_access" } }
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
  stdout { codec => rubydebug }
}
```

We need to have logs that can be imported and manipulated by Logstash. Let's go ahead and download an Apache access log to use. 
```
wget http://bit.ly/logstash-apache -O access_log
```

Now that we have installed and configured Logstash let's go ahead and start it up. 

```
cd /usr/share/logstash/
sudo bin/logstash -f /etc/logstash/conf.d/logstash.conf
```

After starting Logstash wait a couple seconds and you should see it start to index all of the data from our `access_log`.  All of this data is being imported into our Elasticsearch index. 

Logstash normally runs as a daemon monitoring for new data from the `input` systems and then it sends that data over to the `output` systems. 

Confirm the access log data was imported into Elasticsearch successfully. 

List all the indices 
```
curl -XGET 127.0.0.1:9200/_cat/indices?v
```

You should see quite a few new indices with data for each day in our log file. 

```
health status index               uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-2017.05.01 rUEL5_TpQQGIlnTR2rwrGw   5   1      15948            0        7mb            7mb
yellow open   movies              qHJxfatIRlOQzIVMHHXhIA   5   1       9125            0      1.4mb          1.4mb
yellow open   logstash-2015.05.20 w_T-y-3OQNGwXhe0CzTbuQ   5   1       4750            0     22.7mb         22.7mb
yellow open   ratings             jWzRWMcNSbS3H-hz0CyPaQ   5   1     100004            0     14.4mb         14.4mb
```

Look at one of the indices 
```
curl -XGET '127.0.0.1:9200/logstash-2017.05.02/_search?pretty'
```

Notice that all of the data was imported and fields created and indexed successfully. 

## Logstash - MySQL 

Install MySQL first
```
sudo apt-get update 
sudo apt-get install -y mysql-server
```

Now we need to import some data into MySQL. 

Start by changing back to the student home directory
```
cd ~
```

Download the movie data
```
wget http://files.grouplens.org/datasets/movielens/ml-100k.zip
unzip ml-100k.zip 
```

Now look at the format of `ml-100k/u.item` and you'll see it's a pipe delimited file. 

Log into MySQL 
```
sudo mysql -uroot -ppassword
```

Create a new database 
```
CREATE DATABASE movielens;
```

Create a new table
```
CREATE TABLE movielens.movies (
-> movieID INT PRIMARY KEY NOT NULL,
-> title TEXT,
-> releaseDate DATE
-> );
```


Now load the data into the database 
```
LOAD DATA LOCAL INFILE 'ml-100k/u.item' INTO TABLE movielens.movies FIELDS TERMINATED BY '|'
-> (movieID, title, @var3)
-> set releaseDate = STR_TO_DATE(@var3, '%d-%M-%Y');
```

Now if the above commands were all successful you should be able to run the following to confirm all the records were imported. 
```
SELECT * FROM movielens.movies WHERE title like 'Star%';
```

You should get back something similar to
```
+---------+------------------------------------------------+-------------+
| movieID | title                                          | releaseDate |
+---------+------------------------------------------------+-------------+
|      50 | Star Wars (1977)                               | 1977-01-01  |
|      62 | Stargate (1994)                                | 1994-01-01  |
|     222 | Star Trek: First Contact (1996)                | 1996-11-22  |
|     227 | Star Trek VI: The Undiscovered Country (1991)  | 1991-01-01  |
|     228 | Star Trek: The Wrath of Khan (1982)            | 1982-01-01  |
|     229 | Star Trek III: The Search for Spock (1984)     | 1984-01-01  |
|     230 | Star Trek IV: The Voyage Home (1986)           | 1986-01-01  |
|     271 | Starship Troopers (1997)                       | 1997-01-01  |
|     380 | Star Trek: Generations (1994)                  | 1994-01-01  |
|     449 | Star Trek: The Motion Picture (1979)           | 1979-01-01  |
|     450 | Star Trek V: The Final Frontier (1989)         | 1989-01-01  |
|    1068 | Star Maker, The (Uomo delle stelle, L') (1995) | 1996-03-01  |
|    1265 | Star Maps (1997)                               | 1997-01-01  |
|    1293 | Star Kid (1997)                                | 1998-01-16  |
|    1464 | Stars Fell on Henrietta, The (1995)            | 1995-01-01  |
+---------+------------------------------------------------+-------------+
15 rows in set (0.00 sec)
```

Now we need to create a new `jdbc` user to connect to MySQL

Run the following in MySQL 
```
CREATE USER 'jdbc'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'jdbc'@'%';
FLUSH PRIVILEGES;
```

Confirm you can log into mysql as the `jdbc` user

```
mysql -ujdbc -ppassword
```

Logstash is a Java application and to connect to MySQL is requires the `jdbc` driver.  Installing this is fairly simple. 

Download the latest connector and unzip it.

```
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.zip
unzip mysql-connector-java-5.1.46.zip
```

Now we need to configure Logstash to connect to MySQL. 

Create a new configuration file for Logstash `/etc/logstash/conf.d/mysql.conf` with the following data.
```
input {
    jdbc {
        jdbc_connection_string => "jdbc:mysql://localhost:3306/movielens"
        jdbc_user => "jdbc"
        jdbc_password => "password"
        jdbc_driver_library => "/home/student/mysql-connector-java-5.1.46/mysql-connector-java-5.1.46-bin.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        statement => "SELECT * FROM movies"
    }
}

output {
    stdout { codec => json_lines }
    elasticsearch {
        "hosts" => "localhost:9200"
        "index" => "movielens-sql"
        "document_type" => "data"
    }
}
```

Now we need to restart Logstash and pass it the new mysql config file. 

```
cd /usr/share/logstash/
sudo bin/logstash -f /etc/logstash/conf.d/mysql.conf
```


Now let's confirm the data was imported correctly. 

```
curl -XGET '127.0.0.1:9200/movielens-sql/_search?q=title:Star&pretty'
```

# Lab Complete 
