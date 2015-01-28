---
layout: default
title: Content Recommendation Models
---

# Content Recommendation Models
Seldon provides various content recommendation models you can train from your user and item data. We will describe here each type of model, its pros and cons, how its trained and how to deploy it to the live Seldon API server.

For the Seldon VM We assume you will be using one of the pre-configured Seldon clients, e.g., CLIENT1. In the examples that follow replace ${CLIENT} with the desired client.

Each type of model has configuration parameters that need to be chosen to create a good model. In future releases of Seldon we will provide both offline and online optimization methods to find the best parameters for your data and live activity. For the time being you will need to test various models to find the best settings for your use case.

# Quick Start

 1. Load your data into Seldon as describe [HERE]()
 1. Run create_models for the models you want to create
 {% highlight bash %}
    dist/tools/create_models.sh ${CLIENT} ${MODELS}
 {% endhighlight %}

 e.g.

 {% highlight bash %}
    dist/tools/create_models.sh client1 matrix_factorization
 {% endhighlight %}



# Generic Configuration Options
The Seldon models are stored in a directory structure that allows easy integration into a production environment where models are created periodically, usually each day. The directory structure if of the form
 {% highlight bash %}
    seldon-models/${CLIENT}/${MODEL}/${DAY}
 {% endhighlight %}
	
e.g. for a matrix_factorization model created for client client1 created on 27 Jan 2014 (unix epoch day 16461) would be 

 {% highlight bash %}
    seldon-models/client1/matrix_factorization/14232
 {% endhighlight %}

 The configuration for a model captures this information with
 
 * start_day : the start day to use as input and the day which will be used as output of any model created.
 * num_days : the number of days of data to use back in time starting from start_day (inclusive).
 * base_input_folder : the input folder to find the data. Will be expanded as above with client, model and day values.
 * base_output_folder : the output folder to write the model. Will be expanded as above with client, model and day values.

An example:

 {% highlight json %}
   {
   "start_day":1, 
   "num_days":1, 
   "base_input_folder":"local://seldon-models", 
   "base_output_folder":"local://seldon-models"
   }
  {% endhighlight %}	


# Matrix Factorization
An algorithm made popular due to its sucess in the Netflix competition. It tries to find a small set of latent user and item factors that explain the user-item viewing data. We use the [Apache Spark ALS](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) implementation.

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

 1. Set the configuration options in consul, e,g,
 {% highlight bash %}
    docker exec consul curl -s -X PUT -d 
    '{"start_day":1, 
       "num_days":1, 
       "rank":30, 
       "lambda":0.1, 
       "alpha":1, 
       "iterations":5,
       "base_input_folder":"local://seldon-models", 
       "base_output_folder":"local://seldon-models"}' 
   "http://localhost:8500/v1/kv/seldon/${CLIENT}/algs/matrix_factorization"
  {% endhighlight %}	
 1. run the model creation process
 {% highlight bash %}
    docker exec -it spark_offline_server_container bash -c "/spark-jobs/matrix-factorization.sh ${CLIENT}"
  {% endhighlight %}	
# Item Similarity

# Tag Similarity (Semantic Vectors)

# Word2Vec

# Topic Model
