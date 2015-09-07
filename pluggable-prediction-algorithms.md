---
layout: default
title: Pluggable Prediction Algorithms
---

# Microservice Runtime Predictive Services

Microservice runtime prediction services allow you to add any custom predictive scorer into Seldon rather than being limited to the ones available in the core Java server.  The following sections describe the various components:

 * [Offline model creation](#offline-model)
 * [Online external prediction](#online-predictive-scoring)
   * [Microservices REST API](#prediction-internal-rest-api)
   * [Zookeeper configuration](#prediction-zookeeper-conf)
 * [Prepackaged Microservices](#prepackaged)
   * [Vowpal Wabbit predictive scorer](#prepackaged-vw)
   * [XGBoost predictive scorer](#prepackaged-xgboost)
 * [Creating your own Predictive Scorer](#create-your-own)
   * [Example python predictive scoring template](#prediction-python-template)
   * [Example python predictive scoring with Vowpal Wabbit](#prediction-python-vw)

## Offline Model<a name="offline-model"></a>

You can utilize any method to create the offline model. You can use the seldon '''/events''' endpoints in the [REST](api-oauth.html#events) and [javascript](api-javascript.html#events) APIs to inject the events. Some examples of creating models with Vowlpal Wabbit and XGBoost are [provided](offline-prediction-models.html).



## Online Predictive Scoring<a name="online-predictive-scoring"></a>

For the online predictive scoring of an external algoithm we provide a REST API definition that any external predictive scoring algorithm must conform to. You would create a component that satisfies this REST API and publish its endpoint within the Seldon zookeeper configuration for the client you want to have use it. These steps are explained below. Finally, we have provided a python reference template that satisfies this REST API that you can use to write your own external recommender along with an example interface to use Vowpal Wabbit as the online predictive scorer.

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



# Prepackaged Predictive microservices<a name="prepackaged"></a>
Seldon provides prepackaged microservices using our [python package](python-package.html). At present this allows one to build VW and XGBoost models and runtime predictive scorers.

## Vowpal Wabbit Predictive Scorer<a name="prepackaged-vw"></a>
Create a Vowpal Wabbit runtime scorer that will run a feature extraction pipeline and perform the predictions on a loaded model.

`docker pull seldonio/vw_runtime`

This image contains a python script `/vw/vw_runtime/setup.py` which download the models a initialises the config needed for the server. Then a vw daemon is started and a HTTP server to server requests. The setup entry script has the following arguments:

 * **--client** : client
 * **--inputPath** : input base folder to find features data
 * **--day** : days to get features data for
 * **--zkHosts** : zookeeper
 * **--awsKey** : aws key - needed if input or output is on AWS and no IAM
 * **--awsSecret** : aws secret - needed if input or output on AWS  and no IAM
 * **--namespaces** :  JSON providing per feature namespace mapping - default is no namespaces
 * **--include** : include these features
 * **--exclude** : exclude these features

An example using the iris data set can be found in `external/predictor/python/docker/examples/iris`. Run `make vw_runtime` to run all tasks including model building and feature extraction and start a vw microservice to server requests. The command to run the vw microservice runtime scorer is shown below.

{% highlight bash %}
docker run --name="vw_runtime" -d -v ${PWD}/data:/data -p 5000:5000 seldonio/vw_runtime bash -c "cd /vw/vw_runtime;python setup.py --client iris --day 1 --inputPath /data --exclude svmfeatures ;./start_vw_service.sh"
{% endhighlight %}

Run, `make test_vw_server` to test the server which has ports exposed on your localhost at port 5000. This will run:

{% highlight bash %}
curl -G  "http://127.0.0.1:5000/predict?client=iris" --data-urlencode 'json={"f1": 4.6, "f2": 3.2, "f3": 1.4, "f4": 0.2}'
{% endhighlight %}

The response should be:

{% highlight json %}
{
  "predictions": [
    {
      "confidence": 1.0, 
      "predictedClass": "Iris-virginica", 
      "prediction": 0.1820051759757194
    }, 
    {
      "confidence": 1.0, 
      "predictedClass": "Iris-setosa", 
      "prediction": 0.5443882835483635
    }, 
    {
      "confidence": 1.0, 
      "predictedClass": "Iris-versicolor", 
      "prediction": 0.27360654047591704
    }
  ]
}
{% endhighlight %}


## XGBoost Predictive Scorer<a name="prepackaged-xgboost"></a>
Create an XGBoost runtime scorer that will run a feature extraction pipeline and perform the predictions on a loaded model.

`docker pull seldonio/xgboost_runtime`

This image contains a python script `/vw/vw_runtime/setup.py` which download the models a initialises the config needed for the server. Then an HTTP server to server requests is started. The setup entry script has the following arguments:

 * **--client** : client
 * **--inputPath** : input base folder to find features data
 * **--day** : days to get features data for
 * **--zkHosts** : zookeeper
 * **--awsKey** : aws key - needed if input or output is on AWS and no IAM
 * **--awsSecret** : aws secret - needed if input or output on AWS  and no IAM
 * **--svmFeatures** : the JSON feature containing the svm features

An example using the iris data set can be found in `external/predictor/python/docker/examples/iris`. Run `make xgboost_runtime` to run all tasks including model building and feature extraction and start a xgboost microservice to server requests. The command to run the xgboost microservice runtime scorer is shown below.

{% highlight bash %}
docker run --name="xgboost_runtime" -d -p 5001:5000 -v ${PWD}/data:/data seldonio/xgboost_runtime bash -c "cd xgboost/xgboost_runtime ; python setup.py --client iris --day 1 --inputPath /data --svmFeatures svmfeatures ; ./start-service.sh"
{% endhighlight %}

Run, `make test_xgboost_server` to test the server which has ports exposed on your localhost at port 5001. This will run:

{% highlight bash %}
curl -G  "http://127.0.0.1:5001/predict?client=iris" --data-urlencode 'json={"f1": 4.6, "f2": 3.2, "f3": 1.4, "f4": 0.2}'
{% endhighlight %}

The response should be:

{% highlight json %}
{
  "predictions": [
    {
      "confidence": 1.0, 
      "predictedClass": "Iris-virginica", 
      "prediction": 0.1725599765777588
    }, 
    {
      "confidence": 1.0, 
      "predictedClass": "Iris-setosa", 
      "prediction": 0.4815942049026489
    }, 
    {
      "confidence": 1.0, 
      "predictedClass": "Iris-versicolor", 
      "prediction": 0.17391864955425262
    }
  ]
}
{% endhighlight %}


# Creating your own microservice predicitve scorer<a name="create-your-own"></a>
You can use the existing prepackaged python Docker predictive scorers as a base to make a new predictve scorer. Alternatively, you can follow the steps below which describe a simple predictive scorer that does not transform the input features using a feature extraction pipeline.

## External Python Prediction Template<a name="prediction-python-template"></a>

We have provided a template for writing an external predictive scorer in python.

To use this predictor, follow these steps:

1. Install python dependencies.

        pip install Flask
        pip install gunicorn

1. Clone the Seldon Server project.

        git clone https://github.com/SeldonIO/seldon-server.git

1. Navigate to the python predictor template scripts.

        cd seldon-server/external/predictor/python/template

1. Copy or edit the "**example_predict.py**" script, and customize the "**get_predictions**" function and if needed the "**init**" function.

    The parameters to the "**get_predictions**" function are as follows:

    * **client** : a string for the client
    * **json** : a dictionary representation of the json input

    Expand the "**get_predictions**" function to return a a list of 3-tuples of (score,classId,confidence) of type (double,string,double)

    The parameters to the "**init**" function are as follows:

    * **config** : dictionary from the load of "**server_config.py**", see below

    Add to the init function anything your predictive scoring algorithm needs to setup. For example, the location of a model may be taken from the config and loaded into memory.

1. Modify the script "**server_config.py**".  
    Example:

        PREDICTION_ALG="example_predict"

    * **PREDICTION_ALG** : The name of the script with the custom "get_predictions" function, without the .py extension.

1. Serve the predictive scoring for testing.

        python server.py

1. Serve the predictions using gunicorn for handling concurrent requests.  
    Example

        gunicorn -w 4 -b 127.0.0.1:5000 server:app


###  External Python Prediction Server using Vowpal Wabbit<a name="prediction-python-vw"></a>

We provide an example interface to the popular online machine learning tool [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki) to serve predictions. We create a very simple model from the [Iris dataset](https://archive.ics.uci.edu/ml/datasets/Iris) and get VW to serve this using its daemon functionality. We extend the python template for the Seldon micro-service prediction API to connect to the VW daemon to get recommendations.



To use this predictor, follow these steps:

1. Install python dependencies.

        pip install Flask
        pip install gunicorn

1. Download and install [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki) and ensure VW is in your PATH

1. Clone the Seldon Server project.

        git clone https://github.com/SeldonIO/seldon-server.git

1. Create the VW model and start a daemon

        cd seldon-server/external/predictor/python/vw/iris
        make daemon

1. Navigate to python server folder for VW

        cd seldon-server/external/predictor/python/vw

1. Serve the predictive scoring for testing.

        python server.py

1. Serve the predictions using gunicorn for handling concurrent requests.  
    Example

        gunicorn -w 4 -b 127.0.0.1:5000 server:app

1. Test internal micro-service API via curl

        curl 'http://127.0.0.1:5000/predict?json=\{"f1":1.6,"f2":2.7,"f3":5.3,"f4":1.9\}'

You should see a response like:

{% highlight json %}
{
  "predictions": [
    {
      "confidence": 1.0,
      "predictedClass": "Iris-setosa",
      "prediction": 0.12470537784315563
    },
    {
      "confidence": 1.0,
      "predictedClass": "Iris-versicolor",
      "prediction": 0.37473067225490664
    },
    {
      "confidence": 1.0,
      "predictedClass": "Iris-virginica",
      "prediction": 0.5005639499019378
    }
  ]
}
{% endhighlight %}

To integrate this into Seldon server you will need to follow next steps:

1. [Setup Seldon Server](/seldon-server-setup.html) 

1. Set configuration.
  
        cd seldon-server/scripts/zookeeper
        echo 'set /config/default_prediction_strategy {"algorithms":[{"name":"externalPredictionServer","config":[{"name":"io.seldon.algorithm.external.url","value":"http://127.0.0.1:5000/predict"}]}]}' | python set-client-config.py --zookeeper localhost

1. Test prediction, replace CONSUMER_KEY with the key setup when you configured the Seldon server:

        curl  'http://127.0.0.1:8080/seldon-server/js/predict?consumer_key=CONSUMER_KEY&jsonpCallback=a&f1=1.6&f2=2.7&f3=5.3&f4=1.9'

You should receive a response like:

{% highlight json %}
a(
	{"size":3,
	"requested":0,
	"list":[
		{"prediction":0.12470537784315563,"predictedClass":"Iris-setosa","confidence":1.0},
		{"prediction":0.37473067225490664,"predictedClass":"Iris-versicolor","confidence":1.0},
		{"prediction":0.5005639499019378,"predictedClass":"Iris-virginica","confidence":1.0}
		]
	}
)
{% endhighlight %}