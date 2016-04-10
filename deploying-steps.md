---
layout: default
title: Content Recommendation Guide
---

# Content Recommendation 
This guide takes you through the steps to set up Seldon to serve content recommendations.

 * [Create client and meta-data schema](#client)
 * [Import historical data](#historical)
 * [Ingest activity via API](#api)
 * [Create a recommendation model](#model)
 * [Configure runtime recommendation scoring](#runtime)
 * [Server recommendations](#recommendations)
 * [Worked example](#example)
 * [Advanced settings](#advanced)

A worked example using the [Movielens 100K](http://grouplens.org/datasets/movielens/100k/) dataset is also provided [here](ml100k.html).

# Create client and meta-data schema<a name="client"></a>
To serve content recommendation you first need to create a client which will have an associated consumer key.

You can create this via the seldon CLI as describe [here](seldon-cli#client).

Next you will need to define the item meta-data schema. Here is an example schema for an item representing a music album:

{% highlight javascript %}
	{
    "types": [{
            "type_id": 1,
            "type_name": "music",
            "type_attrs": [
                {"name":"title","value_type":"string"},
                {"name":"artist","value_type":"string"},
                {"name":"genre","value_type":["pop","rock","rap"]},
                {"name":"price","value_type":"double"},
                {"name":"is_compilation","value_type":"boolean"},
                {"name":"sales_count", "value_type":"int"}
                ]
            }
            ]
	}
{% endhighlight %}

Let's go though these fields one by one

 1. type_id: Distinguishes between different types of items for example movies and music. For now, Seldon only allows one item type so this must always be 1
 1. type_name: unique name for this type of item
 1. type_attrs: a list of attributes that can be associated with this item type
 1. type_attrs -> name: the name of the attribute in question
 1. type_attrs -> value_type: what type of data to expect for this attribute. Valid values are 'string', 'text', 'double', 'datetime', 'boolean', 'int' and a list. The list is a special case where the data can be one of a restricted list (an enum essentially)

One you have created the attributes JSON file you can associate it to your client using the [seldon CLI](seldon-cli.html#attr).

# Import historical Data<a name="historical"></a>

If you have historical data you will to use with the built in Seldon Spark jobs then you need to convert it into JSON format with references to the internal IDs used by seldon for the users and items. This can be done using the Seldon CLI as described below..

## Add historical items

Items must be provided as a CSV and conform to the schema with 1 or two extra fields. An example for the schema above is

{% highlight bash %}
id,name,title,artist,genre,price,is_compilation,sales_count
1,"tune1","tune1","artist1","pop",0,1
2,"tune2","tune2","artist2","rock",1,20
3,"tune3","tune3","artist1","rock",1,30
{% endhighlight %}

Note that we have '**id**' and '**name**' that were not mentioned in the schema. 'id' is a required field for all items in all schemas. It can be any unique string and is your identifier for the item. 'name' is optional and should be a string that you might use to search for the item. A few other things to mention here are

 1. Boolean fields (is_compilation for example) can be 0 or 1, 0 meaning false and 1 meaning true.
 1. Enum fields (genre for example) must be one of the values you defined in the schema
 1. You must provide a header line
 1. There is an example in /your_data/items_data/example_items.csv

Load the CSV into the DB using the [Seldon CLI import](/seldon-cli.html#import) command.

## Add historical users

Users are much easier as currently it is not presently possible to specify a schema. So we just need an **id** and optionally a username:

{% highlight bash %}
id,username
1,John
2,Jane
3,Sarah
4,Jim
{% endhighlight %}

Load the CSV into the DB using the [Seldon CLI import](/seldon-cli.html#import) command.

## Add historical actions
If you have existing historical activity data you can import these "actions" into seldon if they can be provided as a CSV file.

Actions again have no schema to contend with but we need a few extra fields:

{% highlight bash %}
user_id,item_id,value,time
1,2,1,1422128735
2,2,1,1422752450
3,3,1,1422735290
4,1,1,1422792312
5,1,1,1422795111
5,3,1,1422754829
{% endhighlight %}

The first two columns should be obvious. 'value' is a field that represents the magnitude of the action. If all actions are created equal, then you should just set this to one. 'time' is the unixtimestamp of the action.

The actions are not added to the DB, but they require transformation so that the Spark jobs can consume them.

Use the [Seldon CLI import](/seldon-cli.html#import) command.

# Ingest activity via API<a name="api"></a>
In production (or if you have no historical data) you would send new user activity and item meta data to Seldon via its [REST and JS API](api.html).

# Create a recommendation model<a name="model"></a>
Recommendation models can be built using an available technology that can be Dockerized and run inside Kubernetes. However, we provide some pre-packaged Spark based models and associated runtime scorers for those models. We also provide a python library which allows you to build and create models and runtime scorers exposed as microservices.

## Built-in Spark models
We provide several Spark based models. At present only a few are fully exposed via the Seldon CLI. 

 * Matrix Factorization : An algorithm made popular due to its sucess in the Netflix competition. It tries to find a small set of latent user and item factors that explain the user-item interaction data. We use a wrapper around the [Apache Spark ALS](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) implementation.  Note, however, for this release we only provide implicit matrix factorization.

Spark models can be run via the [Seldon CLI command model](seldon-cli.html#model)

## Python based models

Models can also be built and packaged via our python library. At present we provide an wrapper for gensim document similarity models. An example using this is described [here](content-recommendation-example.html) which has an associated [Jupyter notebook](https://github.com/SeldonIO/seldon-server/blob/master/python/examples/doc_similarity_reuters.ipynb). The docsim class used is detailed [here](python/seldon.text.html#module-seldon.text.docsim).

# Configure runtime recommendation scoring<a name="runtime"></a>
Once a model is built the final step is to provide a runtime scorer for the model. This can be set via the Seldon CLI. You should choose an associated runtime scorer for your particular model as outlined, so for example if you built a Matrix Factorization model you should use an associated scorer, e.g. recentMfRecommender or mfRecommender. 

## Microservices

If your runtime scrorer will be exposed as an internal microservice you need to package it as a Docker container that exposes the [microservice recommendation API](api-microservices.html). Once done you can start it using ```kubernetes/bin/run_recommendation_microservice.sh``` which takes 4 arguments:

  * A name for the microservice
  * An image to pull that can be run to start the microservice
  * A version for the image
  * A client to connect the microservice to

The script create a Kubernetes deployment for the microservice in ```kubernetes/conf/microservices```. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.


# Serve recommendations<a name="recommendations"></a>
Recommendations can be accessed via the [Seldon API](api.html).

# Advanced Settings<a name="advanced"></a>

# Worked example<a name="example"></a>

A worked example using the [Movielens 100K](http://grouplens.org/datasets/movielens/100k/) dataset is provided [here](ml100k.html).

##  Run A/B Tests
When running multiple recommendation models in production you will want to A/B new models to check they perform better than existing models with live clients before you place them fully into production for all users.

The ability to run A/B and multivariant tests is available within Seldon but not yet exposed via the Seldon CLI. Internally everything is controlled via Zookeeper settings. For those with an understanding of Zookeeper who wish to activiate this functionality can find the details [here](advanced-recommender-config.html#multivariate-tests)

## Combine Multiple Algorithms

In some setting you may wish to combine multipe algorithms together to get a combined result. Currently this is not available in the seldon CLI but those with an understanding of Zookeeper can find the details [here](advanced-recommender-config.html#cascading-algorithms)

## API Controlled Variants

In some more complex content recommendation installation you may want to control for a particualar single client (single API key) various different algorithms for different settings, e.g. provide in-section Sport content recommendations and cross-site general content recommendations in different parts of a web page. The changes need to implement this are discussed [here](advanced-recommender-config.html#recommendation-variants)




