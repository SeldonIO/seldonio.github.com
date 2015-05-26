---
layout: default
title: Offline Prediction Model Creation
---

# Offline Prediction Model Creation

Seldon currently supports models created by Vowpal Wabbit. Simple classification models can be runtime scored inside the [seldon server](runtime-prediction.html) while more complex models can be accessed via our [micro-service API](pluggable-prediction-algorithms.html#prediction-python-vw).

## Create VW Features from JSON Events
Seldon injests data via the /events endpoints in its API. These events are stored as JSON. A simple Spark job is provided to create a Vowpal Wabbit training file from JSON events.

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/vwfeatures
{% endhighlight %}

The algorithm specific parameters are:

 * **targetFeature** : the JSON feature to use as the target
 * **excludedFeatures** : features to ignore from the JSON
 * **oaa** :  whether to create a "one-against-all" model

The data parameters are:
 
 * **inputPath** : location of the actions data
 * **outputPath** : location of the output vw training file and class id map

Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "startDay" : 1,
  "days" : 1,
  "targetFeature" : "iris-name",
  "excludedFeatures" : "client,timestamp"
  "oaa" : true
}
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/vwfeatures {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"targetFeature":"iris-name","excludedFeatures":"client,timestamp","oaa":true,"local":true}
{% endhighlight %}

## Run Modeling

{% highlight bash %}
SELDON_SPARK_HOME=~/seldon-spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-0.93-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
BASE_DIR=~/seldon-models

${SPARK_HOME}/bin/spark-submit \
    --class "io.seldon.spark.vw.CreateVwFeatures" \
    --master local[1] \
    ${JAR_FILE_PATH} \
    --client ${CLIENT} \
    --zookeeper 127.0.0.1 
{% endhighlight %}

