---
layout: default
title: JavaScript API for Prediction
---


# Seldon JavaScript API for Prediction

The Seldon JavaScript API provides the simplest method of integrating Seldon onto web based services. It provides the following methods:

* [Events](#events) : Called to send to Seldon arbitray event data to be used to build a predictive model
* [Prediction](#predictive-scoring) : do a real time prediction based on some features


## Events <a name="events"></a>
An endpoint to allow the injestion of arbitrary JSON event data.

{% highlight http %}
GET     /js/event/new
{% endhighlight %}

Input

* consumer_key 
* json : the url encoded JSON data (if provided this will be assumed to contain the event data)
* jsonpCallback : jsonp callback

If a json parameter is not provied then all parameters will be marshalled into a JSON event other than the reserved parameters: consumer_key, consumer_secret, oauth_token, client and jsonpcallback

If a timestamp field is not provided one will be added.

Example

{% highlight http %}
http://<HOST>/js/event/new?consumer_key=XYZ&user=1&num_rooms=4&postcode=sw1&price=400000&jsonpCallback=j
{% endhighlight %}

Assuming the consumer key matches client "client1" this would create an event object similar to below:

{% highlight json %}
{
	"num_rooms":4,
	"postcode":"sw1",
	"price":400000,
	"client":"client1",
	"timestamp":1421336333669
}
{% endhighlight %}	

## Predict <a name="predictive-scoring"></a>

{% highlight http %}
GET     /js/predict
{% endhighlight %}

Input

* consumer_key 
* json : the url encoded JSON data 
* jsonpCallback : jsonp callback

The JSON input should contain two fields

 * meta : optional meta data associated with the prediction request
 * data : features for the prediction request

The meta data can at present just contain a provided optional prediction id "puid".

Example

A housing price predictor based on features:

{% highlight json %}
{
"meta" :
       {
		"puid" : 1
       },
"data":
	{
		"num_bedrooms"    :        2,
		"detached" 	  : true,
		"postcode"    :        "SW1"
	}
}
{% endhighlight %}

Example

{% highlight http %}
http://<HOST>/js/predict?consumer_key=XYZ&json=%7B%22meta%22%3A%7B%22puid%22%3A%221%22%7D%2C%22data%22%3A%7B%22num_rooms%22%3A2%2C%22detached%22%3Atrue%2C%22postcode%22%3A%22sw1%22%7D%7D&jsonpCallback=j
{% endhighlight %}

A response maybe like the following where the "price" field is predicted:

{% highlight json %}
{
  "meta": {
    "puid": "1",
    "modelName": "model_prices",
    "variation": "default"
  },
"predictions":
	[
	{"prediction":400000,"predictedClass":"1","confidence":1.0}
	]
}
{% endhighlight %}
