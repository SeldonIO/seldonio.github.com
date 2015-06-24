---
layout: default
title: Spark Offline Models
---

# Introduction
 
The Seldon Spark component is used to run [Apache Spark](https://spark.apache.org/) jobs that process logs and build models to be used by the Seldon Server.

 * [Actions Grouping Job](#actions)
 * [Matrix Factorization](#matrix-factorization)
 * [Item Activity Similarity](#item-similarity)
 * [Word2Vec](#word2vec)
 * [User Clusters](#user-clusters)
 * [Tag Affinity](#tag-affinity)
 * [Association Rules](#assoc-rules)
  
# Set Up

## Prerequisites

To use Seldon Spark, the following need to be installed:

* Java (v = 7) (Java 8 will **NOT** work at present)
* Spark (v >= 1.3.0)
* Maven (v >=3)
* Git

## Build the Seldon Spark app jar

{% highlight bash %}
cd ~/
git clone https://github.com/SeldonIO/seldon-server.git
cd seldon-server/offline-jobs/spark
mvn -DskipTests=true clean package
{% endhighlight %}


# Actions Grouping Job<a name="actions"></a>

To provide the core activity data needed by many modelling jobs we provide a job which collects the raw action data collected by the API and outputs into daily folders for each client. The input format is presently JSON.

{% highlight json %}
    {
	 "client": "myapp",
	 "client_itemid": "http://myapp.com/somepage",
	 "client_userid": "nveof-3242-ssggsvvf",
	 "itemid": 2301,
	 "rectag": "default",
	 "timestamp_utc": "1998-12-02T05:56:13Z",
	 "type": 1,
	 "userid": 69878,
	 "value": 2.0
	}
{% endhighlight %}

The predicitve modelling jobs that utilize activity will read these processed logs in and output models that use the internal user and item ids. The runtime Seldon server will request scoring using these internal ids. The client user and item ids are the client facing friendly ids and can be arbitary strings, e.g. cookie-ids for users and URLs for items.

An example of running the group actions job.  
The options to the job are as follows:  

* **--input-path-pattern** - This reflects the source of the actions and how Seldon Server was setup to store them. The pattern is based on the path to the location, %y for year, %m for month, %d for day.  
* **--input-date-string** - This is the day that will be processed in unix epoch time.
* **--output-path-dir** - This should be the place the other jobs will use as input for their processing.
* **--gzip-output** - Use this option to compress the output data.

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
DATE_YESTERDAY=$(perl -e 'use POSIX;print strftime "%Y%m%d",localtime time-86400;')
INPUT_DATE_STRING=${DATE_YESTERDAY}
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark

INPUT_DIR=~/seldon-logs
OUTPUT_DIR=~/seldon-models

${SPARK_HOME}/bin/spark-submit \
    --class "io.seldon.spark.actions.GroupActionsJob" \
    --master local[1] \
    ${JAR_FILE_PATH} \
        --aws-access-key-id "" \
        --aws-secret-access-key "" \
        --input-path-pattern "${INPUT_DIR}/actions.%y/%m%d/*/*" \
        --input-date-string "${INPUT_DATE_STRING}" \
        --output-path-dir "${OUTPUT_DIR}" \
        --gzip-output
{% endhighlight %}


# Matrix Factorization<a name="matrix-factorization"></a>
An algorithm made popular due to its sucess in the Netflix competition. It tries to find a small set of latent user and item factors that explain the user-item interaction data. We use a wrapper around the [Apache Spark ALS](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) implementation.  Note, however, for this release we only provide implicit matrix factorization.

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/matrix-factorization
{% endhighlight %}

The algorithm specific parameters are:

 * **rank** : the number of latent factors in the model.
 * **iterations** : the number of iterations to run the modelling
 * **lambda** :  the regularization parameter in ALS to stop over-fitting 
 * **alpha** : governs the baseline confidence in preference observations

The data parameters are:
 
 * **inputPath** : location of the actions data
 * **outputPath** : location of the output model

Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "startDay" : 1,
  "days" : 1,
  "activate" : true,
  "rank" : 30,
  "lambda" : 0.01,
  "alpha" : 1,
  "iterations" : 5
}
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/matrix-factorization {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"activate":true,"rank":30,"lambda":0.01,"alpha":1,"iterations":5,"local":true}
{% endhighlight %}

## Run Modeling


{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
BASE_DIR=~/seldon-models

${SPARK_HOME}/bin/spark-submit \
    --class "io.seldon.spark.mllib.MfModelCreation" \
    --master local[1] \
    ${JAR_FILE_PATH} \
    --client ${CLIENT} \
    --zookeeper 127.0.0.1 
{% endhighlight %}

You can now utilize a [runtime recommendation algorithm](runtime-recommendation.html#matrix-factorization) with this model.

# Item Activity Similarity<a name="item-similarity"></a>
Item similarity models find correlations in the user-item interactions to find pairs of items that have consistently been viewed together. The underlying algorithm is the [DIMSUM algorithm in Apache Spark 1.2](https://blog.twitter.com/2014/all-pairs-similarity-via-dimsum).

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/similar-items
{% endhighlight %}

The algorithm specific parameters are:

 * **limit** : return only the top N similarities for each item
 * **minUsersPerItem** : The minimum number of users who need to have interacted with an item for it to be included
 * **minItemsPerUser** : The minimum number of items that a user has interacted with for that user to be included
 * **maxUsersPerItem** : The maximum number of items a user has interacted with for that user to be included (useful to remove Bots from activity data)
 * **dimsumThreshold** : A value >= 0 and usually < 1.0. Higher the setting the greater the efficiency but less accurate the result. See the [DIMSUM algorithm](https://blog.twitter.com/2014/all-pairs-similarity-via-dimsum) for further details.
  * **sample** : limit the actions processed to a random subset of all actions. Value 0.0 to 1.0. Default 1.0 (all actions)

Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "startDay" : 1,
  "days" : 1,
  "itemType":-1,
  "limit":100,
  "minItemsPerUser" : 0,
  "minUsersPerItem" : 0,
  "maxUsersPerItem" : 2000000,
  "dimsumThreshold" : 0.1,
  "sample" : 1.0
}
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/similar-items {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"itemType":-1,"limit":100,"minItemsPerUser":0,"minUsersPerItem":0,"maxUsersPerItem":2000000,"dimsumThreshold":0.1,"sample":1.0,"local":true}
{% endhighlight %}


## Run Modelling

Example job execution

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.mllib.SimilarItems" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
       --zookeeper 127.0.0.1 	   
{% endhighlight %}

Once complete the Seldon database needs to be updated with the similarities so they can be served.

You will need to get from local filesystem, AWS S3 or HDFS the output of the job and run a script to convert to SQL and upload to the Seldon DB. An example for the result stored on the local filesystem is shown below:
{% highlight bash %}
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
CLIENT=client
DB_HOST=db
DB_USER=user
DB_PASS=pass
DAY=16491
cat /seldon-models/${CLIENT}/item-similarity/${DAY}/part* | ${SELDON_SPARK_HOME}/scripts/item-similarity/create-sql.py > upload.sql
mysql -h${DB_HOST} -u${DB_USER} -p${DB_PASS} ${CLIENT} < upload.sql 
{% endhighlight %}

You can now utilize a [runtime recommendation algorithm](runtime-recommendation.html#similar-items) with this model.

# Word2Vec<a name="word2vec"></a>
Word2Vec models are deep learning based language models that create vector space representations. Traditionally they are trained on a large text corpus but there has been recent use of training them on session activity data. In this setting a "sentence" is a user session whose "words" are the products/items they have interacted with. By training on large amounts of session data we can create vectors that describe the products and can be used to find similar products. For the word2vec training we use the implementation in [Apache Spark](https://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec).

There are two Spark jobs that need to be run:

 * Create session items from the actions activity data
 * Create word2vec vectors from the session data

## Create Session Data

Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/sessionitems
{% endhighlight %}

The specific parameters are:

Example confguration:

 * **minActionsPerUser** : min number of actions per user
 * **maxActionsPerUser** : max number of actions per user
 * **maxIntraSessionGapSecs** : max number of secs before assume session over, -1 for no limit

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "startDay" : 1,
  "days" : 1,
  "maxIntraSessionGapSecs" : -1,
  "minActionsPerUser" : 0,
  "maxActionsPerUser" : 100000
}
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/sessionitems {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"maxIntraSessionGapSecs":-1,"minActionsPerUser":0,"maxActionsPerUser":100000,"local":true}
{% endhighlight %}




Example job execution

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.topics.SessionItems" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
	   --zookeeper 127.0.0.1 
{% endhighlight %}

## Create Word2Vec Model


Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/word2vec
{% endhighlight %}

The algorithm specific parameters are:

Example confguration:

 * **minWordCount** : min count for a token to be included
 * **vectorSize** : vector size

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "activate":true,
  "startDay" : 1,
  "days" : 1,
  "activate" : true,
  "minWordCount" : 50,
  "vectorSize" : 200
}
{% endhighlight %}


An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/word2vec {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"minWordCount":50,"vectorSize":200,"local":true,"activate":true}
{% endhighlight %}



Example job execution

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.features.Word2VecJob" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
	   --zookeeper 127.0.0.1 
{% endhighlight %}

You can now utilize a [runtime recommendation algorithm](runtime-recommendation.html#word2vec) with this model.

# User Clustering Models<a name="user-clusters"></a>
For news sites or other sites where item churn is rapid and relevancy of items decays fast we can base recommendations on what similar users are interacting with on your site. These models are based on clustering the users against a static set of clusters (e.g., an item taxonomy) or in a free way (e.g. using fuzzy k-means). This implementation is based on the assumption items are added into the Seldon datastore with particular dimensions, e.g. article type - sports, finance, gossip etc. See the [deploying your data section](deploying-your-data.html). The model simply groups users who have a more than average association with any particular dimension, e.g., users who read more sports articles than the average user. Recommendation is done by seeing what similar users are interacting with miniute by minute on the live site. Therefore, live activity data is needed to put this model into production and it can't be used just on historical data. 

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/cluster-by-dimension
{% endhighlight %}

The algorithm specific parameters are:

 * **jdbc** : jdbc url to the seldon database to get dimensions for all items
 * **minActionsPerUser** : min number of actions per user
 * **minClusterSize** : min cluster size
 * **delta** :  min difference in dim percentage for user to be clustered in dimension
 
Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "activate":true,
  "startDay" : 1,
  "days" : 1,
  "activate" : "false",
  "jdbc" : "jdbc:mysql://localhost:3306/client?user=root&characterEncoding=utf8",
  "minActionsPerUser" : 0,
  "delta" : 0.1,
  "minClusterSize" : 200
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/cluster-by-dimension {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"jdbc":"jdbc:mysql://localhost:3306/client?user=root&characterEncoding=utf8","minActionsPerUser":0,"delta":0.1,"minClusterSize":200,"local":true,"activate":true}
{% endhighlight %}



## Run Modelling

Example job execution

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.cluster.ClusterUsersByDimension" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
	   --zookeeper 127.0.0.1 
{% endhighlight %}




You can now utilize a [runtime recommendation algorithm](runtime-recommendation.html#user-clustering) with this model.



# Tag Affinity<a name="tag-affinity"></a>
If the item meta data contains tags then a simple model that associates users to tags they have interacted with more than the average user can be utilized.
The output of this model can be used by a simple recommender that combines the tags and their weights to recommend items that are currently popular that have the users tags.

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/tagaffinity
{% endhighlight %}

The algorithm specific parameters are:

 * **jdbc** : jdbc url to the seldon database to get tags for items
 * **minActionsPerUser** : min number of actions per user
 * **tagAttr** : the name of the attribute for an item that contains the tags
 * **minTagCount** : the minimum number of times a user has interacted with a tag before it is included in the result
 * **minPcIncrease** : the minimum percentage increase over the average that a user has interacted with a tag before it is included
 * **tagFilterPath** : the path to a file containing a set of times. Only tags found in this file will be included.
 
Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "activate":true,
  "startDay" : 1,
  "days" : 1,
  "activate" : "false",
  "jdbc" : "jdbc:mysql://localhost:3306/client?user=root&characterEncoding=utf8",
  "minActionsPerUser" : 0,
  "minTagCount" : 10,
  "minPcIncrease" : 0.2
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/tagaffinity {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"jdbc":"jdbc:mysql://localhost:3306/client?user=root&characterEncoding=utf8","minActionsPerUser":0,"minTagCount":10,"minPcIncrease":0.2,"local":true,"activate":true}
{% endhighlight %}



## Run Modelling

Example job execution

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.tags.UserTagAffinity" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
	   --zookeeper 127.0.0.1 
{% endhighlight %}


You can now utilize a [runtime recommendation algorithm](runtime-recommendation.html#tag-based) with this model.


# Association Rules<a name="assoc-rules"></a>
Association rule learning is a form of basket analysis that is useful in e-commerce to provide recommendations for which items could be added to a basket given a current set of items. There are two Spark jobs that need to be run consecutively:

 1. Basket Analysis : This will break up the actions event stream and process the *add-to-basket* and *remove-from-basket* events and create a set of session baskets
 1. Associaton Rules : This will process the baskets , find frequent item sets using the [Spark MLib FP Growth algorithm](https://spark.apache.org/docs/latest/mllib-frequent-pattern-mining.html) and create association rules


## Basket Analysis

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/basketanalysis
{% endhighlight %}

The algorithm specific parameters are:


 * **maxIntraSessionGapSecs** : max number of secs allowed to elapse for consecutive actions to be considered part of the same session
 * **minActionsPerUser** : min number of actions per user
 * **maxActionsPerUser** : max number of actions per user
 * **addToBasketType** : the action type which represents an "add to basket" event
 * **removefromBasketType** : the action type which represents a "remove from basket" event
 
Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "activate":true,
  "startDay" : 1,
  "days" : 1,
  "maxIntraSessionGapSecs" : 3600,
  "minActionsPerUser" : 0,
  "addToBasketType" : 1,
  "removefromBasketType" : 2
}
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/basketanalysis {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"maxIntraSessionGapSecs":3600,"minActionsPerUser":0,"addToBasketType":1,"removefromBasketType":2}

{% endhighlight %}

## Run Modelling

Example job execution

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.transactions.BasketAnalysis" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
	   --zookeeper 127.0.0.1 
{% endhighlight %}


## Association Rules 

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/fpgrowth
{% endhighlight %}

The algorithm specific parameters are:

 * **minSupport** : percentage of total transactions for an itemset to be included
 * **minConfidence** : the min percentage for the itemset as compared to its antecedent for the rule itemset, itemset+item -> item
 * **minInterest** : confidence minus support for item to be recommendde
 * **minLift** : min lift for rule to be included
 * **minItemSetFreq** : min absolute freq of an itemset to be included
 * **maxItemSetSize** :  max length of an itemset to be included
 
Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "activate":true,
  "startDay" : 1,
  "days" : 1,
  "minSupport" : 0.01,
  "minConfidence" : 0.2,
  "minInterest" : 0.1,
  "minLift" : 0.0,
  "maxItemSetSize" : 4
}
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/fpgrowth {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"minSupport":0.01,"minConfidence":0.2,"minInterest":0.1,"minLift":0.0,"maxItemSetSize":4}


{% endhighlight %}

## Run Modelling

Example job execution

{% highlight bash %}
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.transactions.FPGrowthJob" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
	   --zookeeper 127.0.0.1 
{% endhighlight %}

You can now utilize a [runtime recommendation algorithm](runtime-recommendation.html#assoc-rules) with this model.



 