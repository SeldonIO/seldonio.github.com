---
layout: default
title: General Predictive Pipelines
---

# General Prediction Overview

Seldon provides the ability to build supervised learning based predictive models and put them into production at scale. The strutcture of a general predictive pipeline is show in the diagram below:

![Predictive Data Pipelines](/img/predictive-data-pipelines.png)

There are offline and realtime components. Data to be predicted on is sent to Seldon via its REST API. This data is sent as arbitrary JSON. It is normally the case that the raw JSON data sent to Seldon is not in the best format to directly build machine learning models. Therefore, in the offline modelling stage the data is first sent through an (optional) set of feature transformations to extract and create appripriate features that are useful for creating predictive models. After these transformation a model can be build to predict some target feature in the data based on the extracted/transformed features. 

At runtime as prediction calls come in the same set of transformations performed offline will need to be repeated to create the same set of final features to test against the model. Once the transformations have been done the features can be scored and a predictive result returned.

## Data transformations

As Seldon receives JSON events they are stored (presently locally or in AWS S3) in folders of the form:

{% highlight bash %}
/<client>/events/<day_number>/events.json
e.g.
/acme/events/16550/events.json
{% endhighlight %}

 * client - is the identifier of the API user (you can have any number of these)
 * events - folder to store raw events
 * day_number - a unix day number to separate events received into days

The offline feature transformations will then take some number of days of events, for example the last 120 days of events, and transform them into a single new file which is stored in a "features" folder under the current day number. Also stored will be the models needed to recreate the transformation at runtime. Example folders are show below:

{% highlight bash %}
/<client>/models/<day_number>/<feature_model>
/<client>/features/<day_number>/features.json
e.g.
/acme/models/16550/1
/acme/models/16550/2
/acme/features/16550/features.json
{% endhighlight %}

Finally some offline modelling witll be performed and the resulting model stored under a folder for client, model type (e.g. vowpal wabbit) and day number.

{% highlight bash %}
/<client>/vw/<day_number>/model
e.g.
/acme/vw/1/model
{% endhighlight %}

## Example

As an example, assume we wish to predict the type of product a person will buy based on their Facebook likes. An example raw events JSON might be

{% highlight json %}
{"productName":"shoes", "likes" : ["1142214","3463634","34643643"]}
{% endhighlight %}

The target feature is the productName and the provided predictive features are just the "likes" feature in this case. We might wish to transform the likes by treating them as words in a document and creating TFIDF features for each. We might also create a unique internal ids for each product to predict which can more easily be used by the machine learning model. After these transforms on many days of events features an example tranformed JSON line might be:

{% highlight json %}
{"productType" : 1, "productName":"shoes", "like_tfidf" : {"1142214":0.1,"3463634":0.25,"34643643":0.15}}
{% endhighlight %}

Associated feature models to allow us to recreate this transform will have been saved. We build our model and then at runtime we might receive features like:

{% highlight json %}
{"likes" : ["111231","3463634","3465436"]}
{% endhighlight %}

We then rerun our transform pipeline on these to create a set of features we can feed to our runtime scorer, this might create:

{% highlight json %}
{"like_tfidf" : {"111231":0.25,"3463634":0.3,"3465436":0.2}}
{% endhighlight %}

We can then eventually feed this to our model scorer and get back a predicted result, something like:

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
}
{% endhighlight %}

In this case the highest scoring class is the product type "shoes".





