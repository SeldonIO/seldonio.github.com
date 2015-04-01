---
layout: default
title: Content Recommendation Models
---

# Content Recommendation Models
The Seldon VM/AMI provides various content recommendation models you can train from your data. We will describe here each type of model, its pros and cons, how its trained and how to deploy it to the Seldon API server.

For the Seldon VM We assume you will be using one of the pre-configured Seldon clients, test1 to test5. In the examples that follow replace ${CLIENT} with the desired client.

Each type of model has configuration parameters that need to be chosen to create a good model. In future releases of Seldon we will provide both offline and online optimization methods to find the best parameters for your data and live activity but for the time being you will need to test various models to find the best settings for your use case.

# Quick Start

 * Go into the "your_data" folder

{% highlight bash %}
cd dist/your_data
{% endhighlight %}
 * Load your data into Seldon as described [here](data.html)
 * Run create_models for the models you want to create

{% highlight bash %}
./create_models.sh ${CLIENT} ${MODEL}
{% endhighlight %}

e.g.,

{% highlight bash %}
./create_models.sh test1 matrix_factorization
{% endhighlight %}


 * Go to the API Explorer to test

{% highlight http %}
http://127.0.0.1:8080/swagger/
{% endhighlight %}

The API Explorer will require a consumer key and secret for the OAuth API and a consumer key for the js API. These are client specific and are given below for the five test clients in the VM:

 * Client: **test1**, JS consumer key: **test1js**, Oauth consumer_key: **test1consumer**, Oauth secret **test1secret**
 * Client: **test2**, JS consumer key: **test2js**, Oauth consumer_key: **test2consumer**, Oauth secret **test2secret**
 * Client: **test3**, JS consumer key: **test3js**, Oauth consumer_key: **test3consumer**, Oauth secret **test3secret**
 * Client: **test4**, JS consumer key: **test4js**, Oauth consumer_key: **test4consumer**, Oauth secret **test4secret**
 * Client: **test4**, JS consumer key: **test5js**, Oauth consumer_key: **test5consumer**, Oauth secret **test5secret**


# In Depth Discussion

In the following sections we discuss each model and its particular parameters.

# Matrix Factorization
An algorithm made popular due to its sucess in the Netflix competition. It tries to find a small set of latent user and item factors that explain the user-item interaction data. We use the [Apache Spark ALS](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) implementation.

## Configuration options

The configuration options follow those in the [Spark documentation](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html). Note, however, for this release we only provide implicit matrix factorization.

{% highlight json %}
{
 "rank":30, 
 "lambda":0.1, 
 "alpha":1, 
 "iterations":5,
}
{% endhighlight %}	

 * Set the configuration options in consul, e,g,
 {% highlight bash %}
docker exec consul curl -s -X PUT -d \
 '{"rank":30, "lambda":0.1, "alpha":1, "iterations":5}' \ 
 "http://localhost:8500/v1/kv/seldon/${CLIENT}/algs/matrix_factorization"
{% endhighlight %}
  
 * Run the model creation process

{% highlight bash %}
docker exec -it spark_offline_server_container \
  bash -c "/spark-jobs/matrix-factorization.sh ${CLIENT}"
{% endhighlight %}	

# Item Similarity
Item similarity models find correlations in the user-item interactions to find pairs of items that have consistently been viewed together. The underlying algorithm is the [DIMSUM algorithm in Apache Spark 1.2](https://blog.twitter.com/2014/all-pairs-similarity-via-dimsum). 

The key settings are:

 * sample : limit the actions processed to a random subset of all actions. Value 0.0 to 1.0. Default 1.0 (all actions)
 * limit : return only the top N similarities for each item
 * dimsum_threshold : A value >= 0 and usually < 1.0. Higher the setting the greater the efficiency but less accurate the result. See the [DIMSUM algorithm](https://blog.twitter.com/2014/all-pairs-similarity-via-dimsum) for further details.

Example settings:

 {% highlight json %}
 {
 "sample":0.25,
 "limit":100,
 "dimsum_threshold":0.5
 }
  {% endhighlight %}	

 * Set the configuration in Consul

{% highlight bash %}
docker exec consul curl -s -X PUT -d \
 '{"sample":0.25, "limit":100, "dimsum_threshold":0.5}' \
 "http://localhost:8500/v1/kv/seldon/${CLIENT}/algs/item_similarity"
{% endhighlight %}	

 * Create the model using spark and upload to database

{% highlight bash %}
  docker exec -it spark_offline_server_container bash -c "/spark-jobs/item-similarity.sh ${CLIENT}"
  docker run --name="upload_item_similarity_model" -it --rm \
     --volumes-from seldon-models \
	 --link mysql_server_container:mysql_server \
	 --link consul:consul seldon-tools \
	 bash -c "/seldon-tools/scripts/models/item-similarity/create_sql_and_upload.sh ${CLIENT}"
 {% endhighlight %}
 
Recommendations are provided at run-time by using the recent history of a user to find items they have not interacted with that of are of high similarity.

# Tag Similarity (Semantic Vectors)
Tag similarity works purely on the meta data for the items, e.g. tags, to find items with similar meta data. It can be useful for cold-start situations where there is little or no user actions history. We use the [Semantic Vectors](https://code.google.com/p/semanticvectors/) project that provides fast scalable vector space representations.

The key setting is to specify which attribute or attributes in the database contain the textual meta data to be modelled:

 {% highlight json %}
 {
 "attr_names":"top_tags"
 }
  {% endhighlight %}	

 * Set the configuration in Consul

{% highlight bash %}
docker exec consul curl -X PUT -d \
 '{"attr_names":"top_tags"}' \
 "http://localhost:8500/v1/kv/seldon/${CLIENT}/algs/semantic_vectors"
{% endhighlight %}	

 * Create the models using Semantic Vectors loading the item meta data from the database

{% highlight bash %}
docker run --name="create_semantic_vectors" --rm \
 --volumes-from seldon-models \
 --link mysql_server_container:mysql_server \
 --link consul:consul semantic_vectors_image \
 bash -c "/scripts/run-training-consul.sh ${CLIENT}"
{% endhighlight %}

# Word2Vec
Word2vec is a deep learning inspired language technique. See [here](https://code.google.com/p/word2vec/) for background and original code. We utilize the [Apache Spark version](https://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec). Traditionally, word2vec models are trained on textual corpus data but here we utilize it on the session action data of users, for example the ordered play lists of movies for users on a movie viewing site. We are therefore treating each item (e.g., movie) as a "word" and the "sentences" are the ordered actions (e.g., viewing history of movies).

Word2vec has two settings:

 * min_word_count : the minimum number of times an item must be seen in the actions to be included.
 * vector_size : the dimension of the created vectors

For example:

{% highlight json %}
{
 "min_word_count":50,
 "vector_size":30 }
}
{% endhighlight %}	

 * Set the configuration in Consul

{% highlight bash %}
 docker exec consul curl -s -X PUT -d \
   '{ "min_word_count":50, "vector_size":30 }' \
   "http://localhost:8500/v1/kv/seldon/${CLIENT}/algs/word2vec"
{% endhighlight %}	

 * Create the model by creating a training file of the user sessions, training a word2vec model and converting it to a Semantic Vectors file format for loading in the Seldon API server.

{% highlight bash %}
  docker exec -it spark_offline_server_container bash -c "/spark-jobs/session-items.sh ${CLIENT}"
  docker exec -it spark_offline_server_container bash -c "/spark-jobs/word2vec.sh ${CLIENT}"
  docker run --name="translate_word2vec_model" -it --rm \
    --volumes-from seldon-models  \
	--link mysql_server_container:mysql_server \
	--link consul:consul seldon-tools \
	bash -c "/seldon-tools/scripts/models/word2vec/transformToSV.sh ${CLIENT}"
{% endhighlight %}	





## Advanced Configuration Options
The Seldon models are stored in a directory structure that allows easy integration into a production environment where models are created periodically, usually each day. The directory structure is of the form
 {% highlight bash %}
    seldon-models/${CLIENT}/${MODEL}/${DAY}
 {% endhighlight %}
	
e.g. for a matrix_factorization model created for client client1 on 27 Jan 2014 (unix epoch day 16461) would be 

 {% highlight bash %}
    seldon-models/client1/matrix_factorization/16461
 {% endhighlight %}

 The configuration for a model captures this information with
 
 * start_day : the start day to use as input and the day which will be used as output of any model created.
 * num_days : the number of days of data to use back in time starting from start_day (inclusive).
 * base_input_folder : the input folder to find the data. It will be expanded as above with client, model and day values.
 * base_output_folder : the output folder to write the model. It will be expanded as above with client, model and day values.

An example:

 {% highlight json %}
   {
   "start_day":1, 
   "num_days":1, 
   "base_input_folder":"/seldon-models", 
   "base_output_folder":"/seldon-models"
   }
  {% endhighlight %}	


