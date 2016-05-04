---
layout: default
title: Runtime Item Recommendation Algorithms 
---

# Runtime Item Recommendation Algorithms

At runtime the Seldon server can utilize a set of item recommendation algorithms to provide recommendations. These runtime algorithms are usually based on some offline model.  The list of default algorithms and their associated models are described below.

Many of the content recommendation algorithms use two general ways to provide recommendations :

 1. Use the entire built model to score new items for a user
 1. Use the user's recent item interactions as a basis to score new items for a user. This method is using the users recent interests as a basis for item recommendation. For example, use the last three pages a user has read on a news site as a basis to provide recommendations

As Seldon allows combining algorithms both can be used together to produce a final recommendation result.

The current built in runtime item recommendation algorithms are as follows:

**Runtime Item Recommendation Algorithm** | **Offline Model**
--|--
`mfRecommender` | [matrix factorization model](runtime-recommendation.html#matrix-factorization)
`recentMfRecommender` | [matrix factorization model](runtime-recommendation.html#matrix-factorization)
`itemSimilarityRecommender` | [item activity model](runtime-recommendation.html#similar-items)
`recentItemsRecommender` | [baseline models](runtime-recommendation.html#baseline)
`externalItemRecommendationAlgorithm` | [custom model](#custom)



## Applying a Configuration

Use the [seldon-cli](seldon-cli.html#rec_alg) to apply a configuation to a client.

## Matrix Factorization Models<a name="matrix-factorization"></a>

Algorithms that utilize a matrix factorization based model.

 **Algorithm** : `mfRecommender`  
 **Description** : Utilizes the derived offline latent factors to provide a set of recommendations for a user.   

Example config:

{% highlight json %}
 {
    "defaultStrategy": {
        "algorithms": [
            {
                "config": [],
                "filters": [],
                "includers": [],
                "name": "mfRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
{% endhighlight %}


 **Algorithm** :  `recentMfRecommender`  
 **Description** :  Uses the recent item interactions to create a snapshot latent representation of the user from the item factors and use that to recommend new content.  

Optional config setting for this algorithm are:

 * `io.seldon.algorithm.general.numrecentactionstouse` : integer - number of recent item interactions to use to score against

Example config:

{% highlight json %}
 {
    "defaultStrategy": {
        "algorithms": [
            {
                "config": [
                    {
                        "name": "io.seldon.algorithm.general.numrecentactionstouse",
                        "value": "1"
                    }
                ],
                "filters": [],
                "includers": [],
                "name": "recentMfRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
{% endhighlight %}



## Item Activity Models<a name="similar-items"></a>

Algorithms that utilize activity based item similarity.

 **Algorithm** : `itemSimilarityRecommender`   
 **Description** :  Uses the recent item interactions to find a set of top scoring similar items to recommend

Example config:

{% highlight json %}
{
    "defaultStrategy": {
        "algorithms": [
            {
                "config": [],
                "filters": [],
                "includers": [],
                "name": "itemSimilarityRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
{% endhighlight %}



## Baseline Models<a name="baseline"></a>

A set of baseline algorithms used for testing and as final algorithms to use if all other algorithms fail to provide recommendations for a user.

**Algorithm** : `recentItemsRecommender`  
**Description** : Return the most recently added items, e.g., in a news setting the most recent published articles.

Example config:

{% highlight json %}
{
    "defaultStrategy": {
        "algorithms": [
            {
                "config": [],
                "filters": [],
                "includers": [],
                "name": "recentItemsRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
{% endhighlight %}

## Custom Model<a name="custom"></a>

Seldon allows you to deploy your own recommendation runtime scorer using its microservices as described [here](api-microservices.html#content-recommendation)

Configuration parameters:

 * `io.seldon.algorithm.external.url` : string - The URL for the microservice
 * `io.seldon.algorithm.external.name` : String - the name to give to the runtime scorer
 * `io.seldon.algorithm.inclusion.itemsperincluder` : integer - the number of recent items to pass to the external scorer for scoring

Example config:

{% highlight json %}
{
    "defaultStrategy": {
        "algorithms": [
            {
                "config": [
                    {
                        "name": "io.seldon.algorithm.inclusion.itemsperincluder",
                        "value": 1000
                    },
                    {
                        "name": "io.seldon.algorithm.external.url",
                        "value": "http://service_hostname:5000/recommend"
                    },
                    {
                        "name": "io.seldon.algorithm.external.name",
                        "value": "external_model_name"
                    }
                ],
                "filters": [],
                "includers": [
                    "recentItemsIncluder"
                ],
                "name": "externalItemRecommendationAlgorithm"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
{% endhighlight %}

Seldon running inside Kubernetes provides a utility script ```kubernetes/bin/run_recommendation_microservice.sh``` to start a recommendation microservice packaged as a Docker container.






