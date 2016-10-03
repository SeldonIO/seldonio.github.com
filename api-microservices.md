---
layout: default
title: Microservices
---

# Introduction
Seldon makes it easy to extend the set of available algorithms. At runtime you can deploy a microservice to serve as a runtime scorer for your model for both content recommendation and general prediction scenarios.

 * [Content Recommendation Microservices](#content-recommendation)
 * [General Prediction Microservices](#prediction)



# Internal Recommendation API<a name="content-recommendation"></a>


## Microservices REST API

{% highlight http %}
GET     /recommend
{% endhighlight %}	

Parameters

 * **client** : client name
 * **user_id** : user id of the user to provide recommendations
 * **recent_interactions** : a list of item ids of recent items the user has interacted with
 * **exclusion_items** : a list of item ids that should be excluded from scoring
 * **data_key** : a key for memcache to get the items to score
 * **limit** : a max number of recommendations to return

The external algorithm serving the request should return JSON with a list of items with scores with the form shown in the example below. The scores should be normalized to the range 0-1 where 1 is best.

{% highlight json %}
{
  "recommended": [
    {
      "item": 32156,
      "score": 1
    },
    {
      "item": 31742,
      "score": 0.9
    }
  ]
}
{% endhighlight %}	

Below is an example internal call the seldon server would make for a client *test1*, for user with id *1* who has recently interacted with content with item ids *16* and *260* requesting recommendations with no items that need to be excluded and a key for memcache to get the items to score which is *RecentItems:test1:0:100000*

{% highlight http %}
GET /recommend?client=test1&user_id=1&recent_interactions=16,260&exclusion_items=&data_key=ce136f0d0fac274a7d353e672bf4b187&limit=30
{% endhighlight %}	

## Python based recommendation microservice<a name="recommender-python"></a>
Recommendation microservices can easily be created in python. The recommender should extend ```seldon.Recommender``` and provide ```load``` and ```save``` methods to store its model. A wrapper is provided to create a Flask based app given a recommender. For example if the recommender model is saved into "recommender_folder" you can start a microservice for this with:

{% highlight python %}
from seldon.microservice import Microservices
m = Microservices()
app = m.create_recommendation_microservice("recommender_folder")
app.run(host="0.0.0.0",port=5000,debug=False)
{% endhighlight %}

# Internal Prediction API<a name="prediction"></a>

## Microservices REST API

{% highlight http %}
GET     /predict
{% endhighlight %}	

Parameters

 * **client** : client name
 * **json** : JSON representation of the features to score

The external algorithm serving the request should return JSON with an array of objects containing the score, classId and confidence. For example:

{% highlight json %}
{
  "predictions": [
    {
      "score": 0.9,
      "classId": "1",
      "confidence":0.7
    },
    {
      "score": 0.1,
      "classId": "2",
      "confidence":0.7
    }
  ]
}
{% endhighlight %}	

Below is an example internal call the seldon server would make for a client test1

{% highlight http %}
GET /predict?client=test1&json=%7B%22f1%22%3A4.8%2C%22f2%22%3A3.0%2C%22f3%22%3A1.4%2C%22f4%22%3A0.1%7D
{% endhighlight %}	


## Python based Predictive microservices<a name="python"></a>
Seldon provides the ability to build predictive microservice using our [python package](python-package.html) and overview of which is given [here](prediction-pipeline.html).

Any Pipeline built using this package can easily be deployed as a microservice as shown below, where we assume a pipeline has been saved to "./pipeline" and we ish to call the loaded model "test_model":

{% highlight python %} 
from seldon.microservice import Microservices
m = Microservices()
app = m.create_prediction_microservice("./pipeline","test_model")
from gevent.wsgi import WSGIServer
http_server = WSGIServer(('', 5000), app)
http_server.serve_forever()
{% endhighlight %}
