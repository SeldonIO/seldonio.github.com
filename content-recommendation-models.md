---
layout: default
title: Content Recommendation Models
---

# Content Recommendation Models
Seldon contains several built in content recommendations models.

We provide several batch and streaming built-in models. 

 * [Matrix Factorization](content-recommendation-models.html#matrix-factorization)
 * [User Clustered Matrix Factorization](content-recommendation-models.html#clustered-matrix-factorization)
 * [Batch Item Similarity](content-recommendation-models.html#batch-item-similarity)
 * [Streaming item Similarity](content-recommendation-models.html#streaming-item-similarity)
 * [Content Similarity](content-recommendation-models.html#content-similarity)
 * [Static Most Popular](content-recommendation-models.html#static-most-popular)
 * [Recent Most Popular](content-recommendation-models.html#recent-most-popular)

## Matrix Factorization<a name="matrix-factorization"></a>
An algorithm made popular due to its sucess in the Netflix competition. It tries to find a small set of latent user and item factors that explain the user-item interaction data. We use a wrapper around the [Apache Spark ALS](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) implementation.  Note, however, for this release we only provide implicit matrix factorization.

#### Model creation
You can create a matrix factorization Kubernetes job for client "test" starting at unix-day 16907 (17th April 2016) as follows:

{% highlight bash %}
cd kubernetes/conf/models
make matrix-factorization DAY=16907 CLIENT=test
{% endhighlight %}

#### Runtime Scoring

[A matrix factorization scorer can be utilized](runtime-recommendation.html).

## User Clustered Matrix Factorization (experimental)<a name="clustered-matrix-factorization"></a>
In situations where the number of user if very high, in the millions or more, then standard matrix factorization above can become computationally expensive when calculating the recommendatons for all users. An alternative is to cluster users inside the latent factor space created by the factorization of the user-item activity matrix. In this way recommendations can be created just for each cluster. This is viable if many users have very similar tastes and can be served identical recommendations.

#### Model creation
We provide a Spark job which builds on the ideas presented [here](https://spark-summit.org/2015-east/wp-content/uploads/2015/03/SSE15-18-Neumann-Alla.pdf). You can create a user clusters matrix factorization Kubernetes job for client "test" starting at unix-day 16907 (17th April 2016) as follows:

{% highlight bash %}
cd kubernetes/conf/models
make matrix-factorization-clusters DAY=16907 CLIENT=test
{% endhighlight %}

#### Runtime Scoring

[A dedicated user clusters matrix factorization scorer can be utilized](runtime-recommendation.html#matrix-factorization-clusters).





## Batch Item Similarity<a name="batch-item-similarity"></a>
Item similarity models find correlations in the user-item interactions to find pairs of items that have consistently been viewed together. The underlying algorithm is the [DIMSUM algorithm in Apache Spark 1.2](https://blog.twitter.com/2014/all-pairs-similarity-via-dimsum).

#### Model creation
You can create an item similarity modelling job for client "test" starting at unix-day 16907 (17th April 2016) as follows:

{% highlight bash %}
cd kubernetes/conf/models
make item-similarity DAY=16907 CLIENT=test
{% endhighlight %}

#### Runtime Scoring

[The item similarity scorer can be utilized](runtime-recommendation.html#similar-items).



## Streaming Item Similarity<a name="streaming-item-similarity"></a>
Batch item similarity is not applicable for domains such as news recommendation where new items that need to be recommended are published in short time frames. In these circumstances we can use an online streaming item similarity techniques that finds similar items through user activity in short time windows (for example every hour). For this we use a technique described in [Estimating Rarity and Similarity over Data Stream Windows, Mayur Datar, S. Muthukrishnan](https://www.cs.rutgers.edu/%7Emuthu/rare.ps) which uses a streaming adaption of min-hashing to provide efficient item similarity over variable time windows.

#### Model creation
The online algorithm is implemented in two jobs.

 * A kafka streams processor which calculates windowed item similarities
 * A database upload processor which inserts the similarities into the seldon client MySQL database.

The jobs can be created for a client test as follows:

{% highlight bash %}
cd kubernetes/conf/models
make streaming-itemsim CLIENT=test
{% endhighlight %}

You should start these two jobs on the cluster, for example:

{% highlight bash %}
kubectl create -f jobs/stream-itemsim-create-test.json
kubectl create -f jobs/stream-itemsim-dbupload-test.json
{% endhighlight %}

#### Runtime Scoring

[The item similarity scorer can be utilized](runtime-recommendation.html#similar-items).


## Content Similarity<a name="content-similarity"></a>
Rather using collaborative filtering technques which try to find similarities in user's activity and alternative technqiue is to use the actual content to find other content of a similar nature. We have python libraries which wrap several technique provided by the [gensim](https://radimrehurek.com/gensim/) document similarity toolkit to provide these. An example demo can be found [here](content-recommendation-example.html)

## Static Most Popular <a name="static-most-popular"></a>
A baseline model is to provide most popular items. We provide a Spark job that will calculate the item popularity and place the results in the Seldon database for use. 

#### Model creation
There are twpo variants. Most Popular which provides the global most popualr items and Most Popualr by Dimension which creates popularity per dimension (e.g. most popular in Sport, most popular in business etc)

To create the Most Popular job:

{% highlight bash %}
cd kubernetes/conf/models
make most-popular DAY=16907 CLIENT=test
{% endhighlight %}

To create the Most Popular by Dimensio:

{% highlight bash %}
cd kubernetes/conf/models
make most-popular-dim DAY=16907 CLIENT=test
{% endhighlight %}

#### Runtime Scoring

There are run time scorers for [Most Popular](runtime-recommendation.html#static-most-popular) and [Most Popular by Dimension](runtime-recommendation.html#static-most-popular-dim).

## Recent Most Popular <a name="realtime-most-popular"></a>
A strong baseline model especially for high churn scenarios such as news sites is to provide recent popular content. We provide a model that counts items in real time as they are interacted with by users and exponentially decays those counts to provide a continually updated view of popular content. The amount of decay can be controlled to reflect the amount of churn and traffic on a site. This model has only a runtime scoring configuration.

#### Runtime Scoring

[The most popular scorer can be utilized](runtime-recommendation.html#most-popular).



