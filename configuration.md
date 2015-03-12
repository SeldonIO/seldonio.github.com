---
layout: default
title: Seldon Configuration
---

# Seldon Configuration
Seldon uses two tools to provide real time configuration 

 1. [Zookeeper](http://zookeeper.apache.org/) - used to store the location of various models and allow the Seldon API Server to notice the availability of new models which need to be loaded.
 1. [Consul](https://consul.io/) - used to store configuration needed for offline training of models.

## Zookeeper Configration

A lot of configuration options are contained in ZooKeeper, below we pick out a few that are important to gain an understanding of Seldon Server and associated components.

### Clients
 
A comma separated list of clients that are having recommendations created should be stored in the `/clients` node. For example:

{% highlight bash %}
 /clients => "test1,test2"
{% endhighlight %}

### Model location
 
Zookeeper is presently used to specify the algorithms that are active for a client along with the location of the model files. The Seldon API server will watch certain nodes in Zookeeper so it can be immediately informed of changes. Algorithms activated within the API server create watches on a core node `/clients/<alg_name>`, e.g. `/clients/mf` (for matrix factorization). This node will have a comma separated list of clients who are running the algorithm for example:

{% highlight bash %}
 /clients/mf  => "test1,test2"
{% endhighlight %}

For each client in the list for an algorithm there will be a related node holding the location of the models for example:

{% highlight bash %}
 /test1/mf  => "/seldon-models/test1/matrix_factorization/1"
{% endhighlight %}

This will allow the Seldon API server to load into memory the models for this client and serve requests for recommendations.
As stated, these values can be dynamically changed to allow the API server to get updated models and activate new clients.
 
### Algorithms

Zookeeper is also used to store the algorithms chosen to provide recommendations for each client. For example, one client may use matrix factorization where as another may use a clustering algorithm. Now that we have the various concepts defined we can look at how they translate into configuration. A client's algorithms are controlled with JSON stored in a ZooKeeper node hierarchy. Unfortunately this currently has to be inputted manually. Below are some important nodes.

 * /config/default_strategy

 The strategy to use if no client specific strategy is defined. Configured in the same way that a specific client one is. Typical input would be

 {% highlight java %}
 {"algorithms":[{"name":"mfRecommender","tag":"MATRIX_FACTOR","includers":["mostPopularIncluder"],"filters":[],"config":{"items_per_includer":200}},{"name":"globalClusterCountsRecommender","tag":"CLUSTER_COUNTS_GLOBAL","includers":[],"filters":[],"config":{"items_per_includer":200}}],"combiner":"firstSuccessfulCombiner"}
 {% endhighlight %}

 * /config/clients
 
 Which clients have a specific strategy defined for them. This is a comma separated list. Typical input would be

 {% highlight bash %}
 mirror,testclient
 {% endhighlight %}

 It is important that a client isn't added to this list before the below node is populated or else there will be an error.

 * /[clientname]/algs

 The strategy to use for this client. Typical input would be

 {% highlight bash %}
 {"algorithms":[{"name":"dynamicClusterCountsRecommender","tag":"CLUSTER_COUNTS_DYNAMIC","includers":["mostPopularIncluder"],"config":{"items_per_includer":200}},{"name":"globalClusterCountsRecommender","tag":"CLUSTER_COUNTS_GLOBAL","includers":[]},{"name":"itemClusterCountsRecommender","tag":"CLUSTER_COUNTS_FOR_ITEM","includers":[]},{"name":"recentItemsRecommender","tag":"RECENT_ITEMS","includers":[]}],"combiner":"firstSuccessfulCombiner"}
 {% endhighlight %}

 Essentially this is a list of algorithm strategies with a combiner on the end. An algorithm strategy comprises, an algorithm spring bean name ("name" which is the camel case version of the name of thealg class), a tag (the legacy name for the algorithm), optionally a set of includers and/or excluders and optionally some config ("config"). The order of these algorithm strategies is priority order.

## Consul Configuration

### Database Configuration
Database settings for each client to allow access to its Seldon database. At present Seldon requires a JDBC database. The virtual machine presently uses MySql. You can set both read and write settings to allow for replication scenarios.

 * database read access
 * key: 
 {% highlight http %}
    /v1/kv/seldon/${CLIENT}/db_read
 {% endhighlight %}
 * example value
 {% highlight json %}
   {
    "host":"mysql_server",
    "username":"root",
    "password":"mypass",
    "jdbc":"jdbc:mysql://mysql_server:3306/test1?characterEncoding=utf8&user=root&password=mypass"
    }
  {% endhighlight %}	

 * database write access
 * key: 
 {% highlight http %}
    /v1/kv/seldon/${CLIENT}/db_write
 {% endhighlight %}
 * example value
 {% highlight json %}
   {
    "host":"mysql_server",
    "username":"root",
    "password":"mypass",
    "jdbc":"jdbc:mysql://mysql_server:3306/test1?characterEncoding=utf8&user=root&password=mypass"
    }
  {% endhighlight %}	

### Model Configuration
Each model with have a set of parameters that need to be set that are appropriate for your data. See the particular sections describing [content recommendation models](content-recommendation-models.html). All models have keys in Consul for their settings of the form:
 {% highlight http %}
    /v1/kv/seldon/${CLIENT}/algs/${MODEL_NAME}
 {% endhighlight %}
For example:
 {% highlight http %}
    /v1/kv/seldon/test/algs/matrix_factorization
 {% endhighlight %}

