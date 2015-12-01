---
layout: default
title: Pluggable Prediction Algorithms
---

##### General Prediction Steps 

 [setup server](/seldon-server-setup.html) --> [events](prediction-api.html) --> [feature extraction pipeline](feature-pipeline.html) --> [runtime scorer](/runtime-prediction.html) --> **microservice scorer** --> [predictions](prediction-api.html)



# Microservice Runtime Predictive Services

Microservice runtime prediction services allow you to add any custom predictive scorer into Seldon rather than being limited to the ones available in the core Java server.  The following sections describe the various components:

 * [Offline model creation](#offline-model)
 * [Online external prediction](#online-predictive-scoring)
   * [Microservices REST API](#prediction-internal-rest-api)
   * [Zookeeper configuration](#prediction-zookeeper-conf)
 * [Python based predictive pipelines](#python)

## Offline Model<a name="offline-model"></a>

You can utilize any method to create the offline model. You can use the seldon '''/events''' endpoints in the [REST](api-oauth.html#events) and [javascript](api-javascript.html#events) APIs to inject the events. Example using python pipelines are show [below](#python).


## Online Predictive Scoring<a name="online-predictive-scoring"></a>

For the online predictive scoring of an external algoithm we provide a REST API definition that any external predictive scoring algorithm must conform to. You would create a component that satisfies this REST API and publish its endpoint within the Seldon zookeeper configuration for the client you want to have use it. These steps are explained below. Finally, we have provided a python reference template that satisfies this REST API that you can use to write your own external prediction service along with an example interface to use Vowpal Wabbit as the online predictive scorer.

### Microservices REST API<a name="prediction-internal-rest-api"></a>

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


### Zookeeper configuration<a name="prediction-zookeeper-conf"></a>
When you have an external recommendation server running that supports the internal REST API you can activate this new recommender inside Seldon for a particular client by specifying an **externalPredictionServer** for the client in its **predict_algs** node, for example to specify an external algorithm running at ```http://127.0.0.1:5000/predict``` for client **client1** you would set the node ```/all_clients/client1/predict_algs``` to 

{% highlight bash %}
{
"algorithms":
	[{"name":"externalPredictionServer",
	"config":[
		   {"name":"io.seldon.algorithm.external.url","value":"http://127.0.0.1:5000/predict"},
		   {"name":"io.seldon.algorithm.external.name","value":"example_alg"}
         	 ]
         }]
}
{% endhighlight %}	

There are two config options that should be set:

 * io.seldon.algorithm.external.url : the endpoint for the REST API 
 * io.seldon.algorithm.external.name : the name of the algorithm (will appear in the logs)



# Python based Predictive microservices<a name="python"></a>
Seldon provides the ability to build predictive microservice using our [python package](python-package.html) and overview of which is given [here](feature-pipeline.html).

Any Pipeline built using this package can easily be deployed as a microservice as shown below, where we assume a pipeline has been saved to "./pipeline" and we ish to call the loaded model "test_model":

{% highlight python %} 
from seldon.microservice import Microservices
m = Microservices()
app = m.create_prediction_microservice("./pipeline","test_model")
app.run(host="0.0.0.0", debug=True)

{% endhighlight %}

An example predictive pipeline and microservice using the iris dataset running inside a docker container is discussed [here](iris-demo.html) 	

