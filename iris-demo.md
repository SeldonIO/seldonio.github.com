---
layout: default
title: Iris Prediction 
---

![Iris](http://archive.ics.uci.edu/ml/assets/MLimages/Large53.jpg)

# Iris Prediction Demo
We provide a demo for creating a multi-class classification predictive endpoint for the classic Iris classification task using the dataset provided [here](http://archive.ics.uci.edu/ml/datasets/Iris).

The steps are:

 1. [Download the static iris data and create JSON events](#events)
 1. [Create predictive pipelines with XGBoost or Vowpal Wabbit](#pipelines)
 1. [Start runtime prediction microservices](#microservices)
 1. [Integrate into Seldon Server](#seldon-server)

The code for creating the models and predictive pipeline can be found in `python/docker/examples/iris`. 

Prerequisites:

 * A *nix based system with some standard tools: make, wget, python
 * Docker
 * A running Seldon server if you wish to do the final Seldon server integeation step

## Create JSON events data<a name="events"></a>
The Iris data is provided as is and we therefore download and create a JSON dataset to allow us to get started easily. Alternatively, we could start the Seldon server and injest the data via the /events endpoint.

Go to `python/docker/examples/iris` and run:

{% highlight bash %}
   make data/iris/events/1/iris.json
{% endhighlight %}

This will download the raw data and convert into JSON placing the JSON data in a seldon structured client/DAY folder.

## Create Machine Learning Pipelines<a name="pipelines"></a>
For the iris dataset we create very simple pipelines that do the following tasks:

 1. Create an id feature from the name feature
 1. Create an SVMLight feature from the four core predictive features (for use by XGBoost)
 1. Build a model using [XGBoost](https://github.com/dmlc/xgboost) or [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit).

Example code for the XGBoost pipeline is shown below:

{% highlight python %}
import sys, getopt, argparse
import seldon.pipeline.basic_transforms as bt
import seldon.pipeline.util as sutl
import seldon.pipeline.auto_transforms as pauto
from sklearn.pipeline import Pipeline
import seldon.xgb as xg
import sys

def run_pipeline(events,models):

    tNameId = bt.Feature_id_transform(min_size=0,exclude_missing=True,zero_based=True,input_feature="name",output_feature="nameId")
    tAuto = pauto.Auto_transform(max_values_numeric_categorical=2,exclude=["nameId","name"])
    xgb = xg.XGBoostClassifier(target="nameId",target_readable="name",excluded=["name"],learning_rate=0.1,silent=0)

    transformers = [("tName",tNameId),("tAuto",tAuto),("xgb",xgb)]
    p = Pipeline(transformers)

    pw = sutl.Pipeline_wrapper()
    df = pw.create_dataframe(events)
    df2 = p.fit(df)
    pw.save_pipeline(p,models)


if __name__ == '__main__':
    parser = argparse.ArgumentParser(prog='xgb_pipeline')
    parser.add_argument('--events', help='events folder', required=True)
    parser.add_argument('--models', help='output models folder', required=True)

    args = parser.parse_args()
    opts = vars(args)

    run_pipeline([args.events],args.models)
{% endhighlight %}

The various pipelines can run as follows

 * Create an XGBoost pipeline : ```make data/iris/xgb_models/1```
 * Create a VW pipeline : ```make data/iris/vw_models/1```

The models for the pipelines are stored in the locations above

# Online Prediction Microservices
Now that we have built various models we can run a realtime predictor as a microservice that will take in raw features, run our saved feature extraction pipeline and pass these features to the runtime model to score returning a result.

The various services for each pipeline can be started as below

 * Run XGBoost microservice : ```make xgboost_runtime```
 * Run VW microservice : ```make vw_runtime```

We can test test the pipelines with:

 * Send an example to XGBoost microservice : ```make test_xgboost_runtime```
 * Send an example to VW microservice : ```make test_vw_runtime```

Which uses curl to fire a JSON test set of feeatures to the microservice, for xgboost microservice running on port 5001 this would be

{% highlight bash %}
curl -G  "http://127.0.0.1:5001/predict?client=iris" --data-urlencode 'json={"f1": 4.6, "f2": 3.2, "f3": 1.4, "f4": 0.2}'
{% endhighlight %}

The response should be like:

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

This shows the predition to be "Iris-setosa".


# Seldon Server Integration<a name="seldon-server"></a>
We can now integrate our microservice(s) into the Seldon server. You will need a running Seldon server.

First add a new client "iris" to the server by editing the server_config.json. 

{% highlight json %}
{
	"db": {
		"servers": [
			{
				"name":"ClientDB",
				"host":"127.0.0.1",
				"port":3306,
				"user":"root",
				"password":"mypass"
			}
		]
	},
	"memcached": {
		"servers": [
			{
				"host":"localhost",
				"port":11211
			}
		]
	},
	"clients": [
		{
			"name":"iris",
			"db":"ClientDB"
		}
	]
}

{% endhighlight %}

Then run the setup script ```python initial_setup.py``` in the ```scripts``` folder. **note the JS consumer key that is provided**

Next update zookeeper with settings for the "iris" client so that prediction requests are sent to the Vowpal Wabbit microservice. Fom inside a zookeeper client run:

{% highlight bash %}
create /all_clients/iris/predict_algs {"algorithms":[{"name":"externalPredictionServer","config":[{"name":"io.seldon.algorithm.external.url","value":"http://127.0.0.1:5000/predict"},{"name":"io.seldon.algorithm.external.name","value":"vw_iris"}]}]}
{% endhighlight %}

You should now be able to call Seldon API predict requests using the JS consumer key provided able.

{% highlight bash %}
curl  -G "http://127.0.0.1:8080/js/predict?consumer_key=<CONSUMER_KEY_HERE>&jsonpCallback=callback" --data-urlencode 'json={"f1": 4.6, "f2": 3.2, "f3": 1.4, "f4": 0.2}'
{% endhighlight %}

This should give a response like:
{% highlight json %}
callback({"size":3,"requested":0,"list":[{"prediction":0.1820051759757194,"predictedClass":"Iris-virginica","confidence":1.0},{"prediction":0.5443882835483635,"predictedClass":"Iris-setosa","confidence":1.0},{"prediction":0.27360654047591704,"predictedClass":"Iris-versicolor","confidence":1.0}]})
{% endhighlight %}
