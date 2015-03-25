---
layout: default
title: Predictive Scoring 
---

# Predictive Scoring Algorithms

At runtime the Seldon server can utilize a set of predictive scoring algorithms to provide recommendations. These predictive scoring algorithms are usually based on some offline model. To configure which algorithm to use please see the [configuration page](configuration.html). The list of default algorithms and their associated models are described below.

Many of the content recommendation algorithms use two general ways to provide recommendations :

 1. Use the entire built model to score new items for a user
 1. Use the user's recent item interactions as a basis to score new items for a user. This method is using the users recent interests as a basis for predictive scoring. For example, use the last three pages a user has read on a news site as a basis to provide recommendations

As Seldon allows combining algorithms both can be used together to produce a final recommendation result.

The current built in predictive scoring algorithms are as follows:

**Predictive Scoring Algorithm** | **Offline Model**
--|--
`mfRecommender` | [matrix factorization model](predictive-scoring.html#matrix-factorization)
`recentMfRecommender` | [matrix factorization model](predictive-scoring.html#matrix-factorization)
`itemSimilarityRecommender` | [item activity model](predictive-scoring.html#similar-items)
`dynamicClusterCountsRecommender` | [user clustering model](predictive-scoring.html#user-clustering)
`itemClusterCountsRecommender` | [user clustering model](predictive-scoring.html#user-clustering)
`itemCategoryClusterCountsRecommender` | [user clustering model](predictive-scoring.html#user-clustering)
`globalClusterCountsRecommender` | [user clustering model](predictive-scoring.html#user-clustering)
`semanticVectorsRecommender` | [content based models](predictive-scoring.html#content-based), [word2vec model](predictive-scoring.html#word2vec)
`recentItemsRecommender` | [baseline models](predictive-scoring.html#baseline)
`mostPopularRecommender` | [baseline models](predictive-scoring.html#baseline)



## Matrix Factorization Models<a name="matrix-factorization"></a>

Algorithms that utilize a [matrix factorization](spark-models.html#matrix-factorization) based model.

 **Algorithm** : `mfRecommender`  
 **Description** : Utilizes the derived offline latent factors to provide a set of recommendations for a user.   

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
 {
  "algorithms":[
   {
   "name":"mfRecommender",
   "filters":[],
   "includers":[],
   "config":[]
   }
  ],
  "combiner":"firstSuccessfulCombiner"
  }
{% endhighlight %}



 **Algorithm** :  `recentMfRecommender`  
 **Description** :  Uses the recent item interactions to create a snapshot latent representation of the user from the item factors and use that to recommend new content.  

Optional config setting for this algorithm are:

 * `io.seldon.algorithm.general.numrecentactionstouse` : integer - number of recent item interactions to use to score against

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
 {
  "algorithms":[
   {
   "name":"recentMfRecommender",
   "filters":[],
   "includers":[],
   "config":[{"name":"io.seldon.algorithm.general.numrecentactionstouse","value":"1"}]
   }
  ],
  "combiner":"firstSuccessfulCombiner"
  }
{% endhighlight %}



## Item Activity Models<a name="similar-items"></a>

Algorithms that utilize activity based [item similarity](spark-models.html#item-similarity)

 **Algorithm** : `itemSimilarityRecommender`   
 **Description** :  Uses the recent item interactions to find a set of top scoring similar items to recommend

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
 {
  "algorithms":[
   {
   "name":"itemSimilarityRecommender",
   "filters":[],
   "includers":[],
   "config":[]
   }
  ],
  "combiner":"firstSuccessfulCombiner"
  }
{% endhighlight %}


## User Clustering Models<a name="user-clustering"></a>

Algorithms that utilize [user clustering](spark-models.html#user-clusters). These algorithms all collect counts for which items are being viewed by clusters of users in real time. These counts are stored in the Seldon datastore and decayed through time. Recommendations are based on top counts weighted by the clusters a user is contained within. Based on [Google's Scalable News Personalization](http://www2007.org/papers/paper570.pdf). Some live activity data is needed to use this algorithm.

 **Algorithm** : `dynamicClusterCountsRecommender`  
 **Description** : recommend presently popular content being interacted with by users in clusters the target user is contained within  

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
{
    "algorithms": [
        {
            "config": [],
            "filters": [],
            "includers": [],
            "name": "dynamicClusterCountsRecommender"
        }
    ],
    "combiner": "firstSuccessfulCombiner"
}
{% endhighlight %}

 **Algorithm** : `itemClusterCountsRecommender`  
 **Description** : recommend content presently being interacted by users in the dimension of the current item the user has just interacted with, e.g., if the current page is a sports page then recommend content that is currently being read by people in the sports cluster. Note this might not be just sports articles.   

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
{
    "algorithms": [
        {
            "config": [],
            "filters": [],
            "includers": [],
            "name": "itemClusterCountsRecommender"
        }
    ],
    "combiner": "firstSuccessfulCombiner"
}
{% endhighlight %}

**Algorithm** : `itemCategoryClusterCountsRecommender`  
 **Description** : recommend content presently popular in the dimension of the current item the user has just interacted with, e.g., if the current page is a sports page then recommend the top sports articles presently being read  

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
{
    "algorithms": [
        {
            "config": [],
            "filters": [],
            "includers": [],
            "name": "itemCategoryClusterCountsRecommender"
        }
    ],
    "combiner": "firstSuccessfulCombiner"
}
{% endhighlight %}

**Algorithm** : `globalClusterCountsRecommender`  
 **Description** : recommend the most popular content overall that is presently being interacted with  

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
{
    "algorithms": [
        {
            "config": [],
            "filters": [],
            "includers": [],
            "name": "globalClusterCountsRecommender"
        }
    ],
    "combiner": "firstSuccessfulCombiner"
}
{% endhighlight %}


## Content Based Models<a name="content-based"></a>

Algorithms that utilize the content meta data of items to find related items. The offline algorithm required for this predictive scoring will be documented soon.

**Algorithm** : `semanticVectorsRecommender`  
**Description** : Score items based on the content similarity to recent items the user has interacted with  

## Word2vec Models<a name="word2vec"></a>

Algorithms that utilize [word2vec](spark-models.html#user-clusters) models for each item to find related items. 

**Algorithm** : `semanticVectorsRecommender`  
**Description** : Score items based on the word2vec similarity to recent items the user has interacted with  

The word2vec model is scored using the semantic vectors online recommender so a Required config option is:

 * `io.seldon.algorithm.semantic.prefix` : word2vec. 

Optional config settings are:

 * `io.seldon.algorithm.general.numrecentactionstouse` : integer - number of recent item interactions to use to score against


Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
 {
  "algorithms":[
   {
   "name":"semanticVectorsRecommender",
   "filters":[],
   "includers":[],
   "config":[{"name":"io.seldon.algorithm.semantic.prefix","value":"word2vec"}]
   }
  ],
  "combiner":"firstSuccessfulCombiner"
  }
{% endhighlight %}

## Baseline Models<a name="baseline"></a>

A set of baseline predictive scoring algorithms used for testing and as final algorithms to use if all other algorithms fail to provide recommendations for a user.

**Algorithm** : `recentItemsRecommender`  
**Description** : Return the most recently added items, e.g., in a news setting the most recent published articles.

**Algorithm** : `mostPopularRecommender`  
**Description** : Return the most popular items






