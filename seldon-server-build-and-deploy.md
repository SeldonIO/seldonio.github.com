---
layout: default
title: Deploy Seldon Server
---

# Deploy Seldon Server

## Prerequisites
 
To run the Seldon Server, we need a number of things present on the machine it is running on

* Java (v >= 7)
* Tomcat (v >= 7)
* Maven (v >=3)
* [td-agent](http://www.fluentd.org/download) (latest)

There also needs to be a ZooKeeper server running along with Memcached and a MySQL server. It's up to you where you place these, but they should be accessible from the server on which you are running Seldon Server.

## The main project

Clone the project [seldon-server](https://github.com/SeldonIO/seldon-server). 

## Configure Zookeeper
Zookeeper is the central location for all settings needed by Seldon. You will need to enter into zookeeper values for the following nodes:

 * `/config/dbcp` : Database connection pool settings. [Details](configuration.html#dbcp)
 * `/config/default_strategy` : Default algorithms to use`. [Details](configuration.html#algorithms)
 * `/all_clients/[clientname]` : Name of connection pool to use for client. [Details](configuration.html#client)
 * `/all_clients/[clientname]/algs` : (Optional - will fall back to `/config/default_strategy`) Algorithms for [clientname]
 * `/all_clients/[clientmame]/offline/[alg_name]` : Settings for offline models [Details](configuration.html#models)

We provide a python utility script to help you set the correct zookeeper settings in seldon-server/scripts/zookeeper.

 * Edit the file example-client-zookeeper-config.txt
 * Assuming you have a local zookeeper you can set the configuration with:

{% highlight bash %}
cat example-client-zookeeper-config.txt | python set-client-config.py --zookeeper localhost
{% endhighlight %}


## The server project

Change directory to `seldon-server/server`.  
Run `mvn clean package`.  
This will create a war file in the target directory. Deploy this to an Apache Tomcat instance (we recommend that you deploy it at ROOT). You should now have a running Seldon Server instance!



