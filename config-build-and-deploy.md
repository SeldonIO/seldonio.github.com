---
layout: default
title: Seldon Algorithm Configuration
---

# Algorithm Configuration

Algorithm configuration is added via ZooKeeper as per the [configuration page](configuration.html).

## Default configuration

Provided below is a default set of algorithm configurations to add. Put this at `/config/default_strategy` 

{% highlight javascript %}
{"algorithms":[{"name":"mfRecommender","tag":"MATRIX_FACTOR","includers":["mostPopularIncluder"],"filters":[],"config":[{"name":"io.seldon.algorithm.inclusion.itemsperincluder","value":200}],{"name":"recentItemsRecommender","tag":"RECENT_ITEMS","includers":[]}],"combiner":"firstSuccessfulCombiner"}
{% endhighlight %}

This provides a base set of algorithms. If you want to alter this then please refer to the [configuration page](configuration.html).

## Activate model for client in Seldon Server

The table below shows the nodes in zookeeper where the list of clients using a model is specified and where the location of a model for a particular client is specified.


| model | client list node | model location node
|:-------------|:-------------|:-------------| 
| [matrix factorization](spark-models.html#matrix-factorization) | `/config/mf` | `/all_clients/<client_name>/mf` |
| [word2vec](spark-models.html#word2vec) | `/config/word2vec`| `/all_clients/<client_name>/word2vec` |
| [user clusters](spark-models.html#user-clusters)  | `/config/usercluster` | `/all_clients/<client_name>/usercluster` |


The output of the item similarity job is presently placed directly in the db so is not controlled in this way.

To activate a matrix factorization client for client `test1` ensure the client name is added e.g.,

 * `/config/mf => some_other_client1,test1,some_other_client2`

The seldon server will look for the model location in:

 * `/all_clients/test1/mf`








