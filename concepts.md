---
layout: default
title: Concepts 
---

## Client

On each Seldon deployment, it is possible to provide recommendations for several totally different data sets (for example, two websites). These data sets are divided by using different clients. Each call to the API has client as one of the attributes. The instructions we've provided so far only allow one client to be set up, but watch these docs for updates on how to extend this.

## Algorithm
 
Algorithms recommend content/items according to the following specification.

{% highlight java %}
ItemRecommendationResultSet recommend(CFAlgorithm options,String client, Long user, int dimensionId, int maxRecsCount, RecommendationContext ctxt, List<Long> recentItemInteractions);
{% endhighlight %}

A couple of things to focus on here are

 1. **CFAlgorithm** a store for legacy options that will eventually be removed
 1. **RecommendationContext** explained below
 1. **recentItemInteractions** another legacy concept that will eventually be rolled into the RecommendationContext
 1. **maxRecsCount** the maximum recommendations that this alg should return. Even if the alg cannot create this many, it should still return as many as possible as they can be used in certain combiner configurations.

All algorithms must conform to the ItemRecommendationAlgorithm interface and be defined spring components. This is to allow easy discovery of all algorithms when the server starts. 
 
## Includer
 
Provides a set of items for the algorithm to recommend against. You can have multiple of these in which case the union of the set of items are recommended against, or combine them with excluders. An example would be the RecentItemsIncluder which includes items that have recently been added to the DB.

## Excluder

Opposite of Includer

## Combiner

A combiner takes the output of multiple algorithms and combines them into one set of recommendations. An example would be the FirstSuccessfulCombiner which takes the output of whichever algorithm
provides the requisite number of recs first and outputs those. Combiners have the power to stop other algorithms being run if they find that there is already enough recs.
 
## Strategy

Describes how to map the client/user pair to a set of Alg/Context pairs. Usually this is a simple mapping -- all users get the same alg/context pair. However, when a test is set up, a user may be
given different pair based on the hash of his username.
 
## RecommendationContext

Store for algorithm options. One option that is common to all algorithms is the **MODE**. The possible values are **INCLUSION**, **EXCLUSION** or **NONE**. **INCLUSION** means that the set of items stored in the context are to be recommended from, **EXCLUSION** means that they are excluded from any recs and **NONE** means that the context has no opinion on the item set to be recommended from -- the alg is free to choose any item.
 
## ClientAlgorithmStore

Store for which of the above concepts to use for each client, also controls testing.

