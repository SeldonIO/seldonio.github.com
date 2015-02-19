---
layout: default
title: Seldon Configuration
---

# Seldon Configuration
Seldon uses two tools to provide real time configuration 

 1. [Zookeeper](http://zookeeper.apache.org/) - used to store the location of various models and allow the Seldon API Server to notice the availability of new models which need to be loaded.
 1. [Consul](https://consul.io/) - used to store configuration needed for offline training of models.

It is planned to phase out Zookeeper and move towards using Consul in isolation in the near future.

## Zookeeper Configration
Zookeeper is presently used to specify the algorithms that are active for a client along with the location of the model files. The Seldon API server will watch certain nodes in Zookeeper so it can be immediately informed of changes. Algorithms activated within the API server create watches on a core node ```/clients/<alg_name>```, e.g. ```/clients/mf``` (for matrix factorization). This node will have a comma separated list of clients who are running the algorithm for example:

{% highlight bash %}
 /clients/mf  => "test1,test2"
{% endhighlight %}

For each client in the list for an algorithm there will be a related node holding the location of the models for example:

{% highlight bash %}
 /test1/mf  => "/seldon-models/test1/matrix_factorization/1"
{% endhighlight %}

This will allow the Seldon API server to load into memory the models for this client and serve requests for recommendations.
As stated, these values can be dynamically changed to allow the API server to get updated models and activate new clients.

## Consul Configuration

### Database Configuration
Database settings for each client to allow access to its Seldon database. At present Seldon requires a JDBC database. The virtual machine presently uses MySql. You can set both read and write settings to allow for replication scenarios.

 * database read access
 * key: 
 {% highlight http %}
    /v1/kv/seldon/${CLIENT}/db_read
 {% endhighlight %}
 * example value
 {% highlight json %}
   {
    "host":"mysql_server",
    "username":"root",
    "password":"mypass",
    "jdbc":"jdbc:mysql://mysql_server:3306/test1?characterEncoding=utf8&user=root&password=mypass"
    }
  {% endhighlight %}	

 * database write access
 * key: 
 {% highlight http %}
    /v1/kv/seldon/${CLIENT}/db_write
 {% endhighlight %}
 * example value
 {% highlight json %}
   {
    "host":"mysql_server",
    "username":"root",
    "password":"mypass",
    "jdbc":"jdbc:mysql://mysql_server:3306/test1?characterEncoding=utf8&user=root&password=mypass"
    }
  {% endhighlight %}	

### Model Configuration
Each model with have a set of parameters that need to be set that are appropriate for your data. See the particular sections describing [content recommendation models](content-recommendation-models.html). All models have keys in Consul for their settings of the form:
 {% highlight http %}
    /v1/kv/seldon/${CLIENT}/algs/${MODEL_NAME}
 {% endhighlight %}
For example:
 {% highlight http %}
    /v1/kv/seldon/test/algs/matrix_factorization
 {% endhighlight %}

