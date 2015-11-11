---
layout: default
title: General Predictive Pipelines
---

# General Prediction Overview

Seldon provides the ability to build supervised learning based predictive models and put them into production at scale. The strutcture of a general predictive pipeline is show in the diagram below:

![Predictive Data Pipelines](/img/predictive-data-pipelines.png)

There are offline and realtime components. Raw data to be used for creating a predictive pipeline is sent to Seldon via its REST API in real time. This data can be sent as arbitrary JSON which allows complete freedom for a client to provide whatever data is available. It is normally the case that this raw JSON data sent to Seldon is not in the best format to directly build machine learning models. Therefore, in the offline modelling stage the data is first sent through an (optional) set of feature transformations to extract and create appropriate features that are useful for creating predictive models. After these transformations a model can be built to predict some target feature in the data based on the extracted/transformed features. 

At runtime as prediction calls come in the same set of transformations performed offline will need to be repeated to create the same set of final features to test against the model. Once the transformations have been done the features can be scored and a predictive result returned to the client in real time.

## Predictive pipelines

As Seldon receives JSON events they are stored (presently locally or in AWS S3) in folders of the form:

{% highlight bash %}
/<client>/events/<day_number>/events.json
e.g.
/acme/events/16550/events.json
{% endhighlight %}

 * client - is the identifier of the API user (you can have any number of these)
 * events - folder to store raw events
 * day_number - a unix day number to separate events received into days

The offline feature transformations will then take some number of days of events, for example the last 120 days of events, and run a predictive pipeline which may contain some feeature transformation and a final modelling stage using some machine learning algorithm. The models created from the transformations and final estimator are stored for later use at runtime. For example a pipeline with a final xgboost model might be stored in the following folder:

{% highlight bash %}
/<client>/xgboost/<day_number>/model
e.g.
/acme/xgboost/1/model
{% endhighlight %}

## Example

As an example, assume we wish to predict the type of product a person will buy based on their Facebook likes. An example raw events JSON might be

{% highlight json %}
{"productName":"shoes", "likes" : ["1142214","3463634","34643643"]}
{% endhighlight %}

The target feature is the productName and the provided predictive features are just the "likes" feature in this case. We might wish to transform the likes by treating them as words in a document and creating TFIDF features for each. We might also create a unique internal ids for each product to predict which can more easily be used by the machine learning model. Finally we might create an XGBoost model.

Associated feature models to allow us to recreate these transforms will be saved. At runtime we might receive features like:

{% highlight json %}
{"likes" : ["111231","3463634","3465436"]}
{% endhighlight %}

We can rerun the transformation and eventually feed this to our model scorer and get back a predicted result, something like:

{% highlight json %}
{
  "predictions": [
    {
      "predictedClass": "shoes", 
      "prediction": 0.9,
      "confidence": 1.0 
    }, 
    {
      "predictedClass": "hats", 
      "prediction": 0.063,
      "confidence": 1.0
    }
  ]
}
{% endhighlight %}

In this case the highest scoring class is the product type "shoes".





