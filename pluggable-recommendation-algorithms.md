---
layout: default
title: Pluggable Item Recommendtion Algorithms
---

# Pluggable Item Recommendation Algorithms

Pluggable Item Recommendation algorithms allow you to add any custom recommendation algorithm into Seldon rather than being limited to the ones available in the existing code. For example, you might want to use Vowpal Wabbit to build a model and provide predictions from that within the Seldon ecosystem. The following sections describe the various components:

 * [Offline model creation](#offline-model)
 * [Online external item recommendation](#online-recommendation-model)
   * [Microservices REST API](#recommender-internal-rest-api)
   * [Zookeeper configuration](#recommender-zookeeper-conf)
   * [Example python recommender template](#recommender-python-template)

## Offline Model<a name="offline-model"></a>

You can utilize any method to create the offline model. For recommendation models however to fit the model into the Seldon framework you need to ensure you use the internal user and item ids as defined within the Seldon database for your users and items. If you are running Seldon in production we provide a Spark job that takes the daily activity data from REST API calls and stores it in per client JSON files as described [here](spark-models.html#actions). This JSON will contain the user and item IDs and can be used for algorithms that use the user activity to derive their models.

If you are building recommendation models using Spark then a good place to start is to look at the code for the existing Spark jobs in [github](https://github.com/SeldonIO/seldon-server/offline-jobs/spark/).

## Online Recommender<a name="online-recommendation-model"></a>
For the online recommendation component of an external algoithm we provide a REST API definition that any external recommendation algorithm must conform to. You would create a component that satisfies this REST API and publish its endpoint within the Seldon zookeeper configuration for the client you want to have use it. These steps are explained below. Finally, we have provided a python reference template that satisfies this REST API that you can use to write your own external recommender.

### Microservices REST API<a name="recommender-internal-rest-api"></a>

{% highlight http %}
GET     /recommend
{% endhighlight %}	

Parameters

 * **client** : client name
 * **user_id** : user id of the user to provide recommendations
 * **recent_interactions** : a list of item ids of recent items the user has interacted with
 * **exclusion_items** : a list of item ids that should be excluded from scoring
 * **data_key** : a key for memcache to get the items to score
 * **limit** : a max number of recommendations to return

The external algorithm serving the request should return JSON with a list of items with scores with the form shown in the example below. The scores should be normalized to the range 0-1 where 1 is best.

{% highlight json %}
{
  "recommended": [
    {
      "item": 32156,
      "score": 1
    },
    {
      "item": 31742,
      "score": 0.9
    }
  ]
}
{% endhighlight %}	

Below is an example internal call the seldon server would make for a client *test1*, for user with id *1* who has recently interacted with content with item ids *16* and *260* requesting recommendations with no items that need to be excluded and a key for memcache to get the items to score which is *RecentItems:test1:0:100000*

{% highlight http %}
GET /recommend?client=test1&user_id=1&recent_interactions=16,260&exclusion_items=&data_key=ce136f0d0fac274a7d353e672bf4b187&limit=30
{% endhighlight %}	


### Zookeeper configuration<a name="recommender-zookeeper-conf"></a>
When you have an external recommendation server running that supports the internal REST API you can activtae this new recommender inside Seldon for a particular client by specifying an **externalItemRecommendationAlgorithm** for the client in its **algs** node, for example to specify an external algorithm running at ```http://127.0.0.1:5000/recommend``` for client **client1** which uses the most recent 10000 items to score you would set the node ```/all_clients/client1/algs``` to 

{% highlight bash %}
{
"algorithms":
	[{"name":"externalItemRecommendationAlgorithm",
	"includers":["recentItemsIncluder"],
	"filters":[],
	"config":[{"name":"io.seldon.algorithm.inclusion.itemsperincluder","value":100000},
		   {"name":"io.seldon.algorithm.external.url","value":"http://127.0.0.1:5000/recommend"},
		   {"name":"io.seldon.algorithm.external.name","value":"example_alg"}]
         }],
"combiner":"firstSuccessfulCombiner"
}
{% endhighlight %}	

There are two config options that should be set:

 * io.seldon.algorithm.external.url : the endpoint for the REST API 
 * io.seldon.algorithm.external.name : the name of the algorithm (will appear in the logs)

### External Python Recommender Template<a name="recommender-python-template"></a>
We have provided a template for writing an external recommender in python.

To use this recommender, follow these steps:

1. Install python dependencies.

        pip install Flask
        apt-get install libmemcached-dev
        pip install pylibmc
        pip install gunicorn

1. Clone the Seldon Server project.

        git clone https://github.com/SeldonIO/seldon-server.git

1. Navigate to the python recommender scripts.

        cd seldon-server/external/recommender/python

1. Copy or edit the "**example_alg.py**" script, and customize the "**get_recommendations**" function and if needed the "**init**" function.

    The paramters to the "**get_recommendations**" function are as follows:

    * **user_id** : a long for the user id
    * **client** : a str for the client
    * **recent_interactions_list** : a list of item ids of recent itms the user has interacted with
    * **data_set** : a set of item ids to score
    * **limit** : an int for the number of recommendations to return

    Expand the "**get_recommendations**" function to return a dictionary of item_id->score for user with id **user_id**.

    The parameters to the "**init**" function are as follows:

    * **mc** : memcache pool from pylibmc
    * **config** : dictionary from the load of "**recommender_config.py**", see below

    Add to the init function anything your recommender needs to setup. For example, the location of a model may be taken from the config and loaded into memory.

1. Modify the script "**recommender_config.py**".  
    Example:

        RECOMMENDER_ALG="example_alg"
        MEMCACHE={
            "servers": ["192.168.59.103:11211"],
            "pool_size" : 1
        }

    * **RECOMMENDER_ALG** : The name of the script with the custom "get_recommendations" function, without the .py extension.
    * **MEMCACHE** : Change to your memcache settings.
        * **servers** : List of host:ip strings of memcache servers
        * **pool_size** : Size of memcache pool, for the process.

1. Serve the recommender for testing.

        python recommender.py

1. Serve the recommender using gunicorn for handling concurrent requests.  
    Example

        gunicorn -w 4 -b 127.0.0.1:5000 recommender:app

