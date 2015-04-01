---
layout: default
title: Seldon Configuration
---

# Seldon Configuration

Seldon uses [Zookeeper](http://zookeeper.apache.org/) for real time configuration. It is used to store the location of various models and allow the Seldon API Server to notice the availability of new models which need to be loaded.

## Zookeeper Configration

A lot of configuration options are contained in ZooKeeper, below we pick out a few that are important to gain an understanding of Seldon Server and associated components.

### Model location
 
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
 
### Algorithms

Zookeeper is also used to store the algorithms chosen to provide recommendations for each client. For example, one client may use matrix factorization where as another may use a clustering algorithm. Now that we have the various concepts defined we can look at how they translate into configuration. A client's algorithms are controlled with JSON stored in a ZooKeeper node hierarchy. Unfortunately this currently has to be inputted manually. Below are some important nodes.

 * /config/default_strategy

 The strategy to use if no client specific strategy is defined. Configured in the same way that a specific client one is. Typical input would be

 {% highlight json %}
 {
 "algorithms":[
   {
   "name":"mfRecommender",
   "includers":["mostPopularIncluder"],
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

### Model Creation

 * /all_clients/[clientname]/offline/[model]

 The settings for offline creation of a model for a particular algorithm for this client. See [offline models](offline-models.html) for further details and examples. 
