---
layout: default
title: Semantic Vectors Models
---

# Semantic Vectors Models

Semantic Vectors is integrated and allows the creation of content based similarity models derived from the meta data of the items. Your datastore will need to have been [setup](deploying-your-data.html) and [loaded](static-activity-data.html) with items and their meta data. 

We provide a Docker Hub image to allow easy creation of models using semantic vectors.

`docker pull seldonio/semantic-vectors-for-seldon`

## Configuration
Set the configuration in zookeeper at node :

{% highlight bash %}
/all_clients/<client>/offline/semvec
{% endhighlight %}

Activate the client for this algorithm. Make sure the client is part of the comma separated list that's running this algorithm.

{% highlight bash %}
/config/svtext <client>,<other-client1>,<other-client2>,...
{% endhighlight %}

The algorithm specific parameters are:

 * **jdbc** : the JDBC setting to connect to the Seldon datastore
 * **tagAttrs** : the attribute names (comma separated) in the Seldon database containing the meta data to use
 * **itemType** : limit the items to this type, default 1
 * **itemLimit** : the maximum number of items to get from the datastore, default -1 is no limit 

Example confguration:

{% highlight json %}
{
  "outputPath":"/seldon-models",
  "startDay" : 1,
  "days" : 1,
  "activate" : true,
  "itemType" : 1,
  "itemLimit" : 10000,
  "tagAttrs" : "tags",
  "jdbc" : "jdbc:mysql://localhost:3306/client?user=root&characterEncoding=utf8"
}
{% endhighlight %}

An example using zookeeper zkCli to create a new confguration for client "client1" is shown below:

{% highlight bash %}
create /all_clients/client1 ""
create /all_clients/client1/offline ""
create /all_clients/client1/offline/semvec {"inputPath":"/seldon-models","outputPath":"/seldon-models","startDay":1,"days":1,"activate":true,"itemType":1,"itemLimit":10000,"tagAttrs":"tags","jdbc":"jdbc:mysql://localhost:3306/client?user=root&characterEncoding=utf8"}
{% endhighlight %}

## Run Modeling

To run the modelling you should run the image with appropriate networking, in this case we use `--net="host"` to use the host network. In the example below zookeeper is running on the local host. The docker container will also need access to the datastore specified in the `jdbc` setting. The example assumes a client called "client1":

`docker run --name "semantic vectors modeling" -rm  --net="host" seldonio/semantic-vectors-for-seldon bash -c "./semvec/semantic-vectors.py --client client1 --zookeeper 127.0.0.1:2181 "`


You can no utilize a [runtime recommendation algorithm](runtime-recommendation.html#content-based) with this model.