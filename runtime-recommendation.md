---
layout: default
title: Runtime Item Recommendation Algorithms 
---

# Runtime Recommendation Scorers

At runtime the Seldon server can utilize a set of item recommendation scorers to provide recommendations. These runtime scorers are usually based on some offline model.  

The current built in runtime scorers are as follows:

**Runtime Scorer** | **Offline Model**
--|--
[Static Matrix Factorization Scorer](runtime-recommendation.html#matrix-factorization-static) | matrix factorization model
[Recent Activity Matrix Factorization Scorer](runtime-recommendation.html#matrix-factorization-recent) | matrix factorization model
[User Clusters Matrix Factorization Scorer](runtime-recommendation.html#matrix-factorization-clusters) | user clusters matrix factorization model
[Item Similarity Scorer](runtime-recommendation.html#similar-items) | item activity model
[Recent Most Popular](runtime-recommendation.html#most-popular) | 
[Recent Activity](runtime-recommendation.html#recent-activity) | 
[Custom Scorer](#custom) | 



## Applying a Configuration

Use the [seldon-cli](seldon-cli.html#rec_alg) to apply a configuation to a client.

## Static Matrix Factorization Scorer<a name="matrix-factorization-static"></a>
Utilizes the derived offline latent factors to provide a set of recommendations for a user.   

Example activation of a runtime scorer for a client ml100k::

{% highlight bash %}
cat <<EOF | seldon-cli rec_alg --action create --client-name ml100k -f -
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
                "name": "mfRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
EOF

seldon-cli rec_alg --action commit --client-name ml100k
{% endhighlight %}


## Recent Activity  Matrix Factorization Scorer<a name="matrix-factorization-recent"></a>
Uses the recent item interactions to create a snapshot latent representation of the user from the item factors and use that to recommend new content.  

Optional config setting for this algorithm are:

 * `io.seldon.algorithm.general.numrecentactionstouse` : integer - number of recent item interactions to use to score against

Example activation of a runtime scorer for a client ml100k:

{% highlight bash %}
cat <<EOF | seldon-cli rec_alg --action create --client-name ml100k -f -
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
EOF

seldon-cli rec_alg --action commit --client-name ml100k
{% endhighlight %}


## User Clusters Matrix Factorization Scorer<a name="matrix-factorization-clusters"></a>
Utilizes the derived offline latent factors to provide a set of recommendations for a user in a cluster.   

Example activation of a runtime scorer for a client ml100k::

{% highlight bash %}
cat <<EOF | seldon-cli rec_alg --action create --client-name ml100k -f -
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
                "name": "mfUserClustersRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
EOF

seldon-cli rec_alg --action commit --client-name ml100k
{% endhighlight %}



## Item Similarity<a name="similar-items"></a>
Uses the recent item interactions to find a set of top scoring similar items to recommend

Example activation of a runtime scorer for a client ml100k:

{% highlight bash %}
cat <<EOF | seldon-cli rec_alg --action create --client-name ml100k -f -
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
                "name": "itemSimilarityRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
EOF

seldon-cli rec_alg --action commit --client-name ml100k
{% endhighlight %}


## Recent Most Popular<a name="most-popular"></a>
This recommender can be used to serve most popular recommendations. It counts which items are being read  with a exponentially decaying score to control how fast old article counts are forgotton.

Example config:

{% highlight bash %}
cat <<EOF | seldon-cli rec_alg --action create --client-name ml100k -f -
{
    "defaultStrategy": {
        "algorithms": [
            {
                 "config": [
                    {
                        "name": "io.seldon.algorithm.clusters.decayratesecs",
                        "value": "10800"
                    },
                    {
                        "name": "io.seldon.algorithm.clusters.usebucketcluster",
                        "value": "true"
                    }
                ],
                "filters": [],
                "includers": [],
                "name": "globalClusterCountsRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner"
    },
    "recTagToStrategy": {}
}
EOF

seldon-cli rec_alg --action commit --client-name ml100k
{% endhighlight %}

The above configuration ensures:

 * the expoential decay is set to 10800 seconds. The time after which the item activity counts have a negligible effect.
 * uses a bucket cluster to ensure every user's activity is caught. 


## Recent Activity<a name="recent-activity"></a>
Return the most recently added items, e.g., in a news setting the most recent published articles.

Example config:

{% highlight bash %}
cat <<EOF | seldon-cli rec_alg --action create --client-name ml100k -f -
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
EOF

seldon-cli rec_alg --action commit --client-name ml100k
{% endhighlight %}

## Custom Scorer<a name="custom"></a>
Seldon allows you to deploy your own recommendation runtime scorer using its microservices as described [here](api-microservices.html#content-recommendation)

Configuration parameters:

 * `io.seldon.algorithm.external.url` : string - The URL for the microservice
 * `io.seldon.algorithm.external.name` : String - the name to give to the runtime scorer
 * `io.seldon.algorithm.inclusion.itemsperincluder` : integer - the number of recent items to pass to the external scorer for scoring

Example config:

{% highlight bash %}
cat <<EOF | seldon-cli rec_alg --action create --client-name ml100k -f -
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
EOF

seldon-cli rec_alg --action commit --client-name ml100k
{% endhighlight %}

Seldon running inside Kubernetes provides a utility script [start-microservice](scripts.html#start-microservice) to start a recommendation microservice packaged as a Docker container.






