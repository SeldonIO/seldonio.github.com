---
layout: default
title: Seldon Server
---

# Introduction
 
The Seldon Server component is responsible for serving recommendations via a REST API. It consumes models produced by the Spark jobs.

# Concepts

## Algorithm
 
Algorithms recommend content/items according to the following specification.

{% highlight java %}
ItemRecommendationResultSet recommend(CFAlgorithm options,String client, Long user, int dimensionId, int maxRecsCount, RecommendationContext ctxt, List<Long> recentItemInteractions);
{% endhighlight %}

A couple of things to focus on here are

 1. **CFAlgorithm** a store for legacy options that will eventually be removed
 1. **RecommendationContext** explained below
 1. **recentItemInteractions** another legacy concept that will eventually be rolled into the RecommendationContext
 1. **maxRecsCount** the maximum recommendations that this alg should return. Even if the alg cannot create this many, it should still return as many as possible as they can be used in certain combiner configurations.

All algorithms must conform to the ItemRecommendationAlgorithm interface and be defined spring components. This is to allow easy discovery of all algorithms when the server starts. 
 
## Includer
 
Provides a set of items for the algorithm to recommend against. You can have multiple of these in which case the union of the set of items are recommended against, or combine them with excluders. An example would be the RecentItemsIncluder which includes items that have recently been added to the DB.

## Excluder

Opposite of Includer

## Combiner

A combiner takes the output of multiple algorithms and combines them into one set of recommendations. An example would be the FirstSuccessfulCombiner which takes the output of whichever algorithm
provides the requisite number of recs first and outputs those. Combiners have the power to stop other algorithms being run if they find that there is already enough recs.
 
## Strategy

Describes how to map the client/user pair to a set of Alg/Context pairs. Usually this is a simple mapping -- all users get the same alg/context pair. However, when a test is set up, a user may be
given different pair based on the hash of his username.
 
## RecommendationContext

Store for algorithm options. One option that is common to all algorithms is the **MODE**. The possible values are **INCLUSION**, **EXCLUSION** or **NONE**. **INCLUSION** means that the set of items stored in the context are to be recommended from, **EXCLUSION** means that they are excluded from any recs and **NONE** means that the context has no opinion on the item set to be recommended from -- the alg is free to choose any item.
 
## ClientAlgorithmStore

Store for which of the above concepts to use for each client, also controls testing.

# Set Up

## Prerequisites
 
To run the Seldon Server, we need a number of things present on the machine it is running on

* Java (v >= 7)
* Tomcat (v >= 7)
* Maven (v >=3)
* [td-agent](http://www.fluentd.org/download) (latest)

There also needs to be a ZooKeeper server running along with Memcached and a MySQL server. It's up to you where you place these, but they should be accessible from the server on which you are running Seldon Server.

## Config project

The project Seldon [seldon-server-config-template](https://github.com/SeldonIO/seldon-server-config-template) provides a template for the configuration required in the Seldon Server. Clone it and edit `server.properties`, filling in the properties to match your set up. Here is what the properties mean:
* **CLIENT_NAME** - in this initial setup doc we only allow one client. This name should be without spaces and without special characters. It is the name you will use in the REST API etc from now on.
* **ZK_HOST** - the host name of the server that contains the zookeeper instance. This will most likely be localhost.
* **MEMCACHED_HOST** - the host name of the server that contains the Memcached instance. This will most likely be localhost.
* **MEMCACHED_PORT** - see above.
* **DB_HOST** - the host name of the MySQL instance.
* **DB_USER** - the user name of the above.
* **DB_PWD** - the password of the above.
* **LOGS_DIR** - where to output the Seldon Server logs. This can be any directory.
* **TD_LOGS_DIR** - where to output the td-agent logs. These are used for the input for the Spark jobs. It is probable that you are running the Spark jobs on the same machine. If so then this can be any directory. If you are running the Spark jobs on a separate machine then you should give a folder that is on a shared drive.

Run `./rewrite_properties.sh` to propagate the properties from `server.properties` into other property files. Then you are ready to build the config set. Run `mvn clean install` to install the config for use by the Seldon Server. Also produced is the td-agent.conf file that needs to replace the default. The location of this file depends on what platform you are runing on, for me it is `/etc/td-agent/td-agent.conf`. Overwrite this file with the one in the config project and restart td-agent.

## The main project

Clone the project [seldon-server](https://github.com/SeldonIO/seldon-server) and run `mvn clean package`. This will create a war file in the target directory. Deploy this to an Apache Tomcat instance (we recommend that you deploy it at ROOT). You should now have a running Seldon Server instance!



