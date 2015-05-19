---
layout: default
title: External Algorithms
---

# External Algorithms

External algorithms allow you to add any custom algorithm into Seldon rather than being limited to the ones available in the existing code. For example, you might want to use Vowpal Wabbit to build a model and provide predictions from that within the Seldon ecosystem. The following sections describe the various components:

 * [Offline model creation](#offline-model)
 * [Online external item recommendation](#online-recommendation-model)
   * [Microservices REST API](#recommender-internal-rest-api)
   * [Zookeeper configuration](#recommender-zookeeper-conf)
   * [Example python recommender template](#recommender-python-template)
 * [Online external prediction](#online-predictive-scoring)
   * [Microservices REST API](#prediction-internal-rest-api)
   * [Zookeeper configuration](#prediction-zookeeper-conf)
   * [Example python predictive scoring template](#prediction-python-template)

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



## Online Predictive Scoring<a name="online-predictive-scoring"></a>

For the online predictive scoring of an external algoithm we provide a REST API definition that any external predictive scoring algorithm must conform to. You would create a component that satisfies this REST API and publish its endpoint within the Seldon zookeeper configuration for the client you want to have use it. These steps are explained below. Finally, we have provided a python reference template that satisfies this REST API that you can use to write your own external recommender along with an example interface to use vowpall wabbit as the online predictive scorer.

### Microservices REST API<a name="prediction-internal-rest-api"></a>

{% highlight http %}
GET     /predict
{% endhighlight %}	

Parameters

 * **client** : client name
 * **json** : JSON representation of the features to score

The external algorithm serving the request should return JSON with an array of objects containing the score, classId and confidence. For example:

{% highlight json %}
{
  "predictions": [
    {
      "score": 0.9,
      "classId": 1,
      "confidence":0.7
    },
    {
      "score": 0.1,
      "classId": 2,
      "confidence":0.7
    }
  ]
}
{% endhighlight %}	

Below is an example internal call the seldon server would make for a client test1

{% highlight http %}
GET /predict?client=test1&json=%7B%22f1%22%3A4.8%2C%22f2%22%3A3.0%2C%22f3%22%3A1.4%2C%22f4%22%3A0.1%7D
{% endhighlight %}	


### Zookeeper configuration<a name="prediction-zookeeper-conf"></a>
When you have an external recommendation server running that supports the internal REST API you can activtae this new recommender inside Seldon for a particular client by specifying an **externalItemRecommendationAlgorithm** for the client in its **algs** node, for example to specify an external algorithm running at ```http://127.0.0.1:5000/recommend``` for client **client1** which uses the most recent 10000 items to score you would set the node ```/all_clients/client1/algs``` to 

{% highlight bash %}
{
"algorithms":
	[{"name":"externalPredictionServer",
	"config":[
		   {"name":"io.seldon.algorithm.external.url","value":"http://127.0.0.1:5000/recommend"},
		   {"name":"io.seldon.algorithm.external.name","value":"example_alg"}]
         	 ]
         }]
}
{% endhighlight %}	

There are two config options that should be set:

 * io.seldon.algorithm.external.url : the endpoint for the REST API 
 * io.seldon.algorithm.external.name : the name of the algorithm (will appear in the logs)


### External Python Prediction Template<a name="prediction-python-template"></a>

We have provided a template for writing an external predictive scorer in python.

To use this predictor, follow these steps:

1. Install python dependencies.

        pip install Flask
        pip install gunicorn

1. Clone the Seldon Server project.

        git clone https://github.com/SeldonIO/seldon-server.git

1. Navigate to the python predictor template scripts.

        cd seldon-server/external/predictor/python/template

1. Copy or edit the "**example_predict.py**" script, and customize the "**get_predictions**" function and if needed the "**init**" function.

    The paramters to the "**get_predictions**" function are as follows:

    * **client** : a string for the client
    * **json** : a dictionary representation of the json input

    Expand the "**get_predictions**" function to return a a list of 3-tuples of (score,classId,confidence) of type (double,integer,double)

    The parameters to the "**init**" function are as follows:

    * **config** : dictionary from the load of "**server_config.py**", see below

    Add to the init function anything your preditive scoring algorithm needs to setup. For example, the location of a model may be taken from the config and loaded into memory.

1. Modify the script "**server_config.py**".  
    Example:

        PREDICTION_ALG="example_predict"

    * **PREDCITION_ALG** : The name of the script with the custom "get_predictions" function, without the .py extension.

1. Serve the predictive scoring for testing.

        python server.py

1. Serve the predictions using gunicorn for handling concurrent requests.  
    Example

        gunicorn -w 4 -b 127.0.0.1:5000 server:app


###  External Python Predcition Server using Vowpal Wabbit

We provide an example interface to the popular online machine learning tool Vowapl Wabbit to server predictions. We create a very simple model from the [Iris dataset](https://archive.ics.uci.edu/ml/datasets/Iris) and get vw to serve this using its daemon functionality. We extend the python template for Seldon micro-service predcition API to connect to the vw daemon to get recommendations.



To use this predictor, follow these steps:

1. Install python dependencies.

        pip install Flask
        pip install gunicorn

1. Download and install [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki)

1. Clone the Seldon Server project.

        git clone https://github.com/SeldonIO/seldon-server.git

1. Create the vw model and start a daemon

        cd seldon-server/external/predictor/python/vw/iris
        make daemon

1. Navigate to python server folder for vw

        cd seldon-server/external/predictor/python/vw

1. Serve the predictive scoring for testing.

        python server.py

1. Serve the predictions using gunicorn for handling concurrent requests.  
    Example

        gunicorn -w 4 -b 127.0.0.1:5000 server:app

1. 
