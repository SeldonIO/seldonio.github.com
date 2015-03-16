---
layout: default
title: Offline Model Creation
---

# Offline Model Creation

Seldon provides a variety of models that can be created and makes it easy for new custom models to be added.

Confguration is either passed on the command line to the offline jobs or set in zookeeper

# Offline Data Store
The Seldon modelling and data manipulation jobs assume a structure for the data storage. This structure allows easy integration into a production environment where models are created periodically, usually each day. The directory structure is of the form
 {% highlight bash %}
    seldon-models/${CLIENT}/${MODEL}/${DAY}
 {% endhighlight %}
	
e.g. for a matrix_factorization model created for client client1 on 27 Jan 2014 (unix epoch day 16461) would be 

 {% highlight bash %}
    seldon-models/client1/matrix_factorization/16461
 {% endhighlight %}

You can use a network file store, AWS S3 or soon HDFS for the actual store.

The jobs that require activity data will use a start day and a number of days to collect from the filesystem the data they need. They will gather data from folders of the form:

{% highlight bash %}
${input-path}/${client}/actions/start-day
${input-path}/${client}/actions/start-day-1
${input-path}/${client}/actions/start-day-2
.
.
${input-path}/${client}/actions/start-day-(num-days)
{% endhighlight %}

For example:

{% highlight bash %}
/seldon-models/client1/actions/16461
/seldon-models/client1/actions/16460
/seldon-models/client1/actions/16459
{% endhighlight %}

The output path will be of the form:

{% highlight bash %}
${output-path}/${client}/${model}/start-day
{% endhighlight %}

For example:

{% highlight bash %}
s3://seldon-models/client1/matrix-factorization/16461
{% endhighlight %}


## Configuration
Configuration is held in zookeeper as JSON in nodes of the form:

{% highlight bash %}
/<client>/offline/<model-name>
{% endhighlight %}

For example:

{% highlight bash %}
/clientname1/offline/similar-items
{% endhighlight %}

All jobs usually have a set of basic parameters they need including

 * **inputFolder** : the base folder on the local file system, S3 or HDFS of the data needed for the job
 * **outputFolder** : the base folder on the local file system, S3 or HDFS where the output will be stored
 * **startDay** : the day as unix epoch day number to start from
 * **days** : the number of days to go back from startDay (inclusive) to collect data as input
 * **awskey** : AWS key (only needed if using S3 for storage)
 * **awssecret** : AWS secret (only needed if using S3 storage)
 * **itemType** : restrict activity data to only these types of items (-1 is allow all)
 
An example:

{% highlight json %}
{
  "inputFolder":"/seldon-models",
  "outputFolder":"/seldon-models",
  "startDay" : 1,
  "days" : 1,
  "awsKey" : "",
  "awsSecret" : "",
  "itemType":-1,
}
{% endhighlight %}


The current integrated models are:

 * [Models created via Apache Spark](spark-models.html)
 * Models created via Semantic Vectors (coming soon)
 * Models created via Vowpal Wabbit (coming soon)


