---
layout: default
title: Pluggable Prediction Algorithms
---

##### General Prediction Steps 

 [setup server](/seldon-server-setup.html) --> [events](prediction-api.html) --> [feature extraction pipeline](feature-pipeline.html) --> [offline model](offline-prediction-models.html) --> [runtime scorer](/runtime-prediction.html) --> **microservice scorer** --> [predictions](prediction-api.html)



# Microservice Runtime Predictive Services

Microservice runtime prediction services allow you to add any custom predictive scorer into Seldon rather than being limited to the ones available in the core Java server.  The following sections describe the various components:

 * [Offline model creation](#offline-model)
 * [Online external prediction](#online-predictive-scoring)
   * [Microservices REST API](#prediction-internal-rest-api)
   * [Zookeeper configuration](#prediction-zookeeper-conf)
 * [Python based predictive pipelines](#python)
 * [Creating your own Predictive Scorer](#create-your-own)
   * [Example python predictive scoring template](#prediction-python-template)
   * [Example python predictive scoring with Vowpal Wabbit](#prediction-python-vw)

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

Any Pipeline built using this package can easily be deployed as a microservice. The code to run a simple Flask server if provided in the Seldon Docker pipelines image. The code is reproduced below

{% highlight python %} 
import sys, getopt, argparse
import importlib
from flask import Flask, jsonify
from flask import request
app = Flask(__name__)
import json
import pprint
from sklearn.pipeline import Pipeline
import seldon.pipeline.util as sutl
import pandas as pd

def extract_input():
    client = request.args.get('client')
    j = json.loads(request.args.get('json'))
    input = {
        "client" : client,
        "json" : j
    }
    return input


@app.route('/predict', methods=['GET'])
def predict():
    print "predict called"
    input = extract_input()
    print input,args.pipeline
    df = pw.create_dataframe(input["json"])
    print df
    preds = pipeline.predict_proba(df)
    print preds
    idMap = pipeline._final_estimator.get_class_id_map()
    formatted_recs_list=[]
    for index, proba in enumerate(preds[0]):
        print index,proba
        if index in idMap:
            indexName = idMap[index]
        else:
            indexName = str(index)
        formatted_recs_list.append({
            "prediction": str(proba),
            "predictedClass": indexName,
            "confidence" : str(proba)
        })
    ret = { "predictions": formatted_recs_list , "model" : args.model_name }
    json = jsonify(ret)
    return json


if __name__ == "__main__":
    parser = argparse.ArgumentParser(prog='microservice')
    parser.add_argument('--model_name', help='name of model', required=True)
    parser.add_argument('--pipeline', help='location of prediction pipeline', required=True)
    parser.add_argument('--aws_key', help='aws key', required=False)
    parser.add_argument('--aws_secret', help='aws secret', required=False)

    args = parser.parse_args()
    opts = vars(args)

    pw = sutl.Pipeline_wrapper(aws_key=args.aws_key,aws_secret=args.aws_secret)
    pipeline = pw.load_pipeline(args.pipeline)

    app.run(host="0.0.0.0", debug=True)
{% endhighlight %}	

 The code:
 
 1. Loads the pipeline from the external location
 1. At predict time 
     1. Translate the JSON into a Pandas DataFrame
     1. Call the pipeline's ```predict_proba``` to get class probabilities 
     1. Use the class_id map (see BasePandasEstimator) to return more friendly results.

# Creating your own microservice predicitve scorer<a name="create-your-own"></a>
Alternatively, you can follow the steps below which describe a simple predictive scorer that can load any python library to satisfy the microservice requirements. This would be appropriate if your internal prediction models could not be packaged as a scikit-learn pipeline.

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


###  External Python Prediction Server using Vowpal Wabbit Daemon<a name="prediction-python-vw"></a>

We provide an example interface to the popular online machine learning tool [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki) to serve predictions. We create a very simple model from the [Iris dataset](https://archive.ics.uci.edu/ml/datasets/Iris) and get VW to serve this using its daemon functionality. We extend the python template for the Seldon micro-service prediction API to connect to the VW daemon to get predictions.



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