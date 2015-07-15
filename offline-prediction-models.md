---
layout: default
title: Offline Prediction Model Creation
---

# Offline Prediction Model Creation

Seldon currently supports models created by [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki). Simple Vowpal Wabbit classification models can be runtime scored inside the [seldon server](runtime-prediction.html) while more complex models can be accessed via our [micro-service API](pluggable-prediction-algorithms.html#prediction-python-vw).


## Create a Vowpal Wabbit model
Create a Vowpal Wabbit model from features. We provide a Docker container to run the modelling:

`docker pull seldonio/vw`

### Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/vw
{% endhighlight %}

The algorithn specific options are:

 * **vwArgs** : the command line options for vw. The job uses wabbit_wappa internally which itself requires "--save_resume --predictions /dev/stdout --quiet" so you should not add these.
 * **features** : Provides a key on how to handle each feature in the input JSON. By default each feature will be converted to feature:value for if the value can be interpreted as a number, otherwise it will be treated as a categorical value and added as feature_value. Two explicit options are presently handled
      * **split** : split the feature value and add each as a separate categorical feature in vw
      * **label** : the specified feature is the target feature, Its value should be an integer.
 * **namespaces** : Specify which namespace each feature should be put in. By default all features are added to the default vw namespace.

An example is: 
{% highlight json %}
{"features":{"f1":"split","t":"label"},"namespaces":{"f1":"words","f2":"f"}}
{% endhighlight %}
For an input:
{% highlight json %}
{"f1":"the cat sat on the mat","f2":2.1,"t":1}
{% endhighlight %}
would create the vw line 
{% highlight bash %}
1 words| the cat sat on the mat f| f2:2.1
{% endhighlight %}


The data parameters are:

 * **client** : the client name 
 * **inputPath** : the base folder for features data
 * **outputPath** : the base folder of the output model
 * **day** : the unix day 

The data will be searched for in:


{% highlight bash %}
<inputPath>/<client>/features/<day>
{% endhighlight %}

The model will be output to

{% highlight bash %}
<outputPath>/<client>/vw/<day>
{% endhighlight %}


Example confguration:

{% highlight json %}
{
  "inputPath":"/seldon-models",
  "outputPath":"/seldon-models",
  "day" : 1,
  "vwArgs" : "--oaa 3",
  "features":{"f1":"split","t":"label"},
  "namespaces":{"f1":"words","f2":"f"}}
}
{% endhighlight %}


## Run Modeling

To run the modelling you should run the image with appropriate networking, in this case we use `--net="host"` to use the host network. In the example below zookeeper is running on the local host. 

`docker run --name "vw modeling" -rm  --net="host" seldonio/vw bash -c "/scripts/vw.py --client client1 --zkHosts 127.0.0.1:2181"`


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
SELDON_VERSION=0.95
SELDON_SPARK_HOME=~/seldon-server/offline-jobs/spark
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-${SELDON_VERSION}-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
BASE_DIR=~/seldon-models

${SPARK_HOME}/bin/spark-submit \
    --class "io.seldon.spark.vw.CreateVwFeatures" \
    --master local[1] \
    ${JAR_FILE_PATH} \
    --client ${CLIENT} \
    --zookeeper 127.0.0.1 
{% endhighlight %}

