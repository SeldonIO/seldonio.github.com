---
layout: default
title: Seldon Configuration
---

# Seldon Configuration

Seldon uses [Zookeeper](http://zookeeper.apache.org/) for real time configuration. It is used to specify all settings needed by the Seldon server as well as offline model creation jobs.

## Zookeeper Configration

The set of available zookeeper configurations settings are shown below.

 * [Memcached](#memcached)
 * [Database pool](#dbcp)
 * [Client datastore](#client)
 * [Model locations](#models)
 * [Client recommendation algorithm settings](#recommender-algorithms)
 * [Client prediction algorithm settings](#prediction-algorithms)
 * [Offline model creation settings](#offline)
 * [Statsd](#statsd)
 * [Airbrake](#airbrake)
 
### Memcached Settings<a name="memcached"></a>
Seldon uses Memcached for caching data. The configuration is set in "**/config/memcached**" example:

{% highlight html %}
{"servers":"192.168.59.103:11211,192.168.59.104:11211","numClients":1}
{% endhighlight %}

Seldon uses the [spymemcached](https://github.com/couchbase/spymemcached) library which has a single I/O thread to each server. It may be useful in high volume settings to increase the number of spymemcached clients. Use the "numClients" feature for this.

### Database Pool Settings<a name="dbcp"></a>
Seldon needs access to a JDBC compliant datastore that holds the client databases as well as a special "api" database that contains the consumer keys and secrets for clients. An [Apache DBCP2](http://commons.apache.org/proper/commons-dbcp/) database pool is configured for each datastore. The configuration is set in `/config/dbcp', an example is show below:

 {% highlight json %}
{"dbs":
  [{
  "name":"ClientDB",
  "jdbc":"jdbc:mysql:replication://host1:3306,host2:3306?characterEncoding=utf8",
  "driverClassName":"com.mysql.jdbc.ReplicationDriver",
  "user":"user",
  "password":"password",
  "maxTotal":600
  "maxIdle":50
  }]
}
 {% endhighlight %}

The possible values follow the availble configuration parameters for Apache DBCP2. The defaults have been set for Mysql Replication driver settings. You will need to modify the settings for your own setup. The full set of settings and defaults are show below:

 {% highlight json %}
{"dbs":
  [{
  "name":"ClientDB",
  "jdbc":"jdbc:mysql:replication://localhost:3306,localhost:3306/?characterEncoding=utf8&useServerPrepStmts=true&logger=com.mysql.jdbc.log.StandardLogger&roundRobinLoadBalance=true&transformedBitIsBoolean=true&rewriteBatchedStatements=true",
  "driverClassName":"com.mysql.jdbc.ReplicationDriver",
  "user":"user1",
  "password":"mypass",
  "maxTotal":600,
  "maxIdle":50,
  "minIdle":20,
  "maxWait":20000,
  "timeBetweenEvictionRunsMillis":10000,
  "minEvictableIdleTimeMillis":60000,
  "testWhileIdle":true,
  "testOnBorrow":true,
  "validationQuery":"/* ping */ SELECT 1",
  "removeAbanadoned":true,
  "removeAbandonedTimeout":60,
  "logAbandonded":false
  }]
}
 {% endhighlight %}

There should always be at least 1 provided datastore with name "ClientDB". The special "api" database catalog should be found in this datastore. See how to set this up [here](db-build-and-deploy.html)

### Client datastore<a name="client"></a>
Each client needs to connect to a datastore which holds the Seldon database for that client. The name of the DBCP datasource to use should be placed in `/all_clients/[clientname]` node in Zookeeper. If there is no value in this node it will try to default to "ClientDB" as the name of the datasource. Example:

{% highlight bash %}
 /all_clients/client1  => "ClientDB"
{% endhighlight %}

### Model location<a name="models"></a>
 
Zookeeper is presently used to specify the algorithms that are active for a client along with the location of the model files. The Seldon API server will watch certain nodes in Zookeeper so it can be immediately informed of changes. Algorithms activated within the API server create watches on a core node `/config/<alg_name>`, e.g. `/config/mf` (for matrix factorization). This node will have a comma separated list of clients who are running the algorithm for example:

{% highlight bash %}
 /config/mf  => "test1,test2"
{% endhighlight %}

For each client in the list for an algorithm there will be a related node holding the location of the models. These are held in nodes under `/all_clients/<client_name>`, for example for a client `test1`:

{% highlight bash %}
 /all_clients/testt1/mf  => "/seldon-models/test1/matrix_factorization/1"
{% endhighlight %}

This will allow the Seldon API server to load into memory the models for this client and serve requests for recommendations.
As stated, these values can be dynamically changed to allow the API server to get updated models and activate new clients.
 
### Recommendation Algorithms<a name="recommender-algorithms"></a>

Zookeeper is also used to store the algorithms chosen to provide recommendations for each client. For example, one client may use matrix factorization where as another may use a clustering algorithm. Now that we have the various concepts defined we can look at how they translate into configuration. A client's algorithms are controlled with JSON stored in a ZooKeeper node hierarchy. Unfortunately this currently has to be inputted manually. Below are some important nodes.

 * /config/default_strategy

 The strategy to use if no client specific strategy is defined. Configured in the same way that a specific client one is. Typical input would be

 {% highlight json %}
 {
 "algorithms":[
   {
   "name":"mfRecommender",
   "includers":["recentItemsIncluder"],
   "filters":[],
   "config":[{"name":"io.seldon.algorithm.inclusion.itemsperincluder","value":200}]
   },
   {
   "name":"globalClusterCountsRecommender",
   "includers":[],
   "filters":[],
   "config":[{"name":"io.seldon.algorithm.inclusion.itemsperincluder","value":200}]
   }
  ],
  "combiner":"firstSuccessfulCombiner"
  }
 {% endhighlight %}

 Essentially this is a list of algorithm strategies with a combiner on the end. An algorithm strategy comprises, an algorithm spring bean name ("name" which is the camel case version of the name of the alg class) and optionally a set of includers and/or excluders and optionally some config ("config"). The order of these algorithm strategies is priority order.

For a client specific algorithm strategy add to 

 * /all_clients/[clientname]/algs

 the strategy to use for this client. 

### Prediction Algorithms<a name="prediction-algorithms"></a>
    
The global default configuration is set in:

 * /config/default_prediction_strategy

An example setting would be:

 {% highlight json %}
{"algorithms":
	[{"name":"externalPredictionServer",
	  "config":[
		{"name":"io.seldon.algorithm.external.url","value":"http://127.0.0.1:5000/predict"}
		]}
	]
}
 {% endhighlight %}

A client specific strategy would be placed in 

 * /all_clients/[clientname]/predict_algs

### Model Creation<a name="offline"></a>

 * /all_clients/[clientname]/offline/[model]

 The settings for offline creation of a model for a particular algorithm for this client. See [offline models](offline-models.html) for further details and examples. 

### Statsd Settings<a name="statsd"></a>
This is **optional** and can be used for gathering stats. The server will check the setting **/config/statsd** and only setup Statsd usage if it exists. The setting is a json object. eg.

{% highlight json %}
{
    "id": "statsTest",
    "port": 8125,
    "sample_rate": 0.25,
    "server": "192.168.59.103"
}
{% endhighlight %}

### Airbrake Settings<a name="airbrake"></a>
This is **optional** and can be used for sending server exception details to the [Airbrake service](https://airbrake.io/). The server will check the setting **/config/airbrake** and only setup Airbrake usage if it exists. The setting is a json object. eg.

{% highlight json %}
{
    "api_key": "_YOUR_AIRBRAKE_KEY_",
    "enabled": true,
    "env": "dev"
}
{% endhighlight %}

