---
layout: default
title: Runtime Item Recommendation Algorithms 
---

# Runtime Item Recommendation Algorithms

At runtime the Seldon server can utilize a set of item recommendation algorithms to provide recommendations. These runtime algorithms are usually based on some offline model. To configure which algorithm to use please see the [configuration page](configuration.html). The list of default algorithms and their associated models are described below.

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
`dynamicClusterCountsRecommender` | [user clustering model](runtime-recommendation.html#user-clustering)
`itemClusterCountsRecommender` | [user clustering model](runtime-recommendation.html#user-clustering)
`itemCategoryClusterCountsRecommender` | [user clustering model](runtime-recommendation.html#user-clustering)
`globalClusterCountsRecommender` | [user clustering model](runtime-recommendation.html#user-clustering)
`semanticVectorsRecommender` | [content based models](runtime-recommendation.html#content-based), [word2vec model](runtime-recommendation.html#word2vec)
`userTagAffinityRecommender` | [tag based model](runtime-recommendation.html#tag-based)
`assocRulesRecommender` | [association rules model](runtime-recommendation.html#assoc-rules)
`recentItemsRecommender` | [baseline models](runtime-recommendation.html#baseline)
`mostPopularRecommender` | [baseline models](runtime-recommendation.html#baseline)
`externalItemRecommendationAlgorithm` | [custom model](pluggable-recommendation-algorithms.html)

These algorithms will be utilized when the item recommendation endpoints of the [REST](api-oauth.html#recommendations) or [Javascript](api-javascript.html#recommendations) API are used.


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

Configuration options required:

 * `io.seldon.algorithm.clusters.minnumberitemsforvalidclusterresult` : minimum number of results to be included as a valid result
 * `io.seldon.algorithm.clusters.decayratesecs` : decay rate in seconds for the counts of what people are reading. Smaller rates will be more reactive but also more noisy. Suggested : 10800
 * `io.seldon.algorithm.clusters.categorydimensionname` : name of the attribute in the Seldon database to be use to used as a dimension to restrict results
 

## Tag Based Models<a name="tag-based"></a>

Algorithms that utilize the content metadata providing tags for item. Building the model is described [here](spark-models.html#tag-affinity).

**Algorithm** : `userTagAffinityRecommender`  
**Description** : Score items based user association with tags, .e.g., a user that has read more than the usual number of articles tagged with "manchester united" would get currently popular articles that have this tag.

Required config settings are:

 * `io.seldon.algorithm.tags.attrid` : integer - attribute id in the Seldon db containing the tags for the items
 * `io.seldon.algorithm.tags.useitemdim` : boolean - whether to use the category dimension of the current page (i.e. to restrict results to the same category as the current item the user is interacting with)

As this recommender using the user clustering real time stats on what people are reading it requires these configuration settings as well:

 * `io.seldon.algorithm.clusters.minnumberitemsforvalidclusterresult` : minimum number of results to be included as a valid result
 * `io.seldon.algorithm.clusters.decayratesecs` : decay rate in seconds for the counts of what people are reading. Smaller rates will be more reactive but also more noisy. Suggested : 10800
 * `io.seldon.algorithm.clusters.categorydimensionname` : name of the attribute in the Seldon database to be use to used as a dimension to restrict results


Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
 {
  "algorithms":[
   {
   "name":"userTagAffinityRecommender",
   "filters":[],
   "includers":[],
   "config":[]
   }
  ],
  "combiner":"firstSuccessfulCombiner"
  }
{% endhighlight %}



## Content Based Models<a name="content-based"></a>

Algorithms that utilize the content metadata of items to find related items. Building the models is described [here](semantic-vectors.html)

**Algorithm** : `semanticVectorsRecommender`  
**Description** : Score items based on the content similarity to recent items the user has interacted with  

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
   "config":[]
   }
  ],
  "combiner":"firstSuccessfulCombiner"
  }
{% endhighlight %}


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


## Association Rules<a name="assoc-rules"></a>

Algorithms that utilize association rules created from offline modeling [here](spark-models.html#assoc-rules).

**Algorithm** : `assocRuleRecommender`  
**Description** : Create recommendations by looking at the current basket of the user and finding the best matching association rules

Required config settings are:


 * `io.seldon.algorithm.assocrules.basket.maxsize` : Integer - the max basket size. At present this has a hard limit of 3 for efficiency reasons
 * `io.seldon.algorithm.assocrules.usetype` : Boolean - whether to use the types of the recent actions to recreate the basket using add and remove action types specified below
 * `io.seldon.algorithm.actions.storefullactions` : Boolean - if `io.seldon.algorithm.assocrules.usetype` is true then you need this set to true as well so the Seldon server will store the full details of each action a user is performing
 * `io.seldon.algorithm.assocrules.add.basket.action.type` : Integer -  the action type that represents "add to basket"
 * `io.seldon.algorithm.assocrules.remove.basket.action.type` : Integer - the action type that represents "remove from basket"

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
{
 "algorithms":[
  {"name":"assocRuleRecommender",
   "includers":[],
   "config":[{"name":"io.seldon.algorithm.assocrules.add.basket.action.type","value":"1"},
		{"name":"io.seldon.algorithm.assocrules.remove.basket.action.type","value":"2"},
		{"name":"io.seldon.algorithm.actions.storefullactions","value":true},
		{"name":"io.seldon.algorithm.assocrules.usetype","value":true}]}],
    "combiner":"firstSuccessfulCombiner"
}
{% endhighlight %}


## Baseline Models<a name="baseline"></a>

A set of baseline algorithms used for testing and as final algorithms to use if all other algorithms fail to provide recommendations for a user.

**Algorithm** : `recentItemsRecommender`  
**Description** : Return the most recently added items, e.g., in a news setting the most recent published articles.

Example config to set in `/all_clients/[client]/algs`:

{% highlight json %}
{
    "algorithms": [
        {
            "config": [],
            "filters": [],
            "includers": [],
            "name": "recentItemsRecommender"
        }
    ],
    "combiner": "firstSuccessfulCombiner"
}
{% endhighlight %}


**Algorithm** : `mostPopularRecommender`  
**Description** : Return the most popular items






