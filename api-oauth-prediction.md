---
layout: default
title: OAuth REST API for Prediction
---

# Seldon REST API for Prediction

The Seldon prediction REST API has a single endpoint which injest arbitrary JSON objects representing feature data to be used to train a model

* [Events](#events) : Arbitrary JSON representing some event from which we want to create a prediction model 

Use the relevant API endpoint to provide predictions

* [Predcitive Scoring](#predictive-scoring)

An API method can be called with the following template
{% highlight http %}
[GET|POST]      /endpoint?oauth_token=t
{% endhighlight %}
	
For security reasons using the HTTPS protocol is recommended.


# Events <a name="events"></a>
Events allow input into Seldon of arbitrary events from which we wish to create a predictive model. 

{% highlight http %}
POST     /events
{% endhighlight %}

Example

The service injects house price data

{% highlight json %}
{
"num_bedrooms"    :        2,
"detached" 	  : true,
"postcode"    :        "SW1",
"price" : 400000
}
{% endhighlight %}


# Prediction (Classification)

{% highlight http %}
POST     /predict
{% endhighlight %}	

The endpoint should be passed JSON containing features from which a prediction is to be made.

Example

A housing price predictor based on features:

{% highlight json %}
{
"num_bedrooms"    :        2,
"detached" 	  : true,
"postcode"    :        "SW1"
}
{% endhighlight %}

Output

{% highlight json %}
{
  "meta": {
    "puid": "f91b158ba046d438cfea82aff4c382f996f5bf51",
    "modelName": "model_prices",
    "variation": "default"
  },
"predictions":
	[
	{"prediction":400000,"predictedClass":"1","confidence":1.0}
	]
}
{% endhighlight %}


