---
layout: default
title: Iris Prediction 
---

![Iris](http://archive.ics.uci.edu/ml/assets/MLimages/Large53.jpg)

# Iris Prediction Demo
We provide a demo for creating a multi-class classification predictive endpoint for the classic Iris classification task using the dataset provided [here](http://archive.ics.uci.edu/ml/datasets/Iris).

The steps are outlined below.

 1. [Download the static iris data and create JSON events](#events)
 1. [Create and run a feature extraction pipleline](#pipeline)
 1. [Create a Vowpal wabbit model](#vw)
 1. [Create an XGBoost model](#xgboost)
 1. [Start a Vowpal Wabbit runtime predcition microservice](#vw-microservice)
 1. [Start an XGBoost runtime prediction microservice](#xgboost-microservice)
 1. [Integrate into Seldon Server](#seldon-server)

The code for creating the models and predictive pipeline can be found in `external/predictor/python/docker/examples/iris`. 

Prerequisites:

 * A *nix based system with some standard tools: make, wget, python
 * Docker
 * A running Seldon server if you wish to do the final Seldon server integeation step

## Create JSON events data<a name="events"></a>
The Iris data is provided as is and we therefore download and create a JSON dataset to allow us to get started easily. Alternatively, we could start the Seldon server and injest the data via the /events endpoint.

Go to `external/predictor/python/docker/examples/iris` and run:

{% highlight bash %}
   make data/iris/events/1/iris.json
{% endhighlight %}

This will download the raw data and convert into JSON placing the JSON data in a seldon structured client/DAY folder. The steps run are:
{% highlight bash %}
	mkdir -p data
	cd data ; wget --quiet http://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data
	mkdir -p data/iris/events/1
	cat data/iris.data | python create-json.py | shuf > data/iris/events/1/iris.json
{% endhighlight %}


## Create Feature Extraction Pipeline<a name="pipeline"></a>
For the iris dataset we need a very simple pipeline that does the following tasks:

 1. Create an id feature from the name feature
 1. Create an SVMLight feature from the four core predictive features (for use by XGBoost)

The code for this is:

{% highlight python %}
import sys, getopt, argparse
import seldon.pipeline.basic_transforms as bt
import seldon.pipeline.pipelines as pl
import sys

def run_pipeline(events,features,models):
    p = pl.Pipeline(input_folders=events,output_folder=features,local_models_folder="models_tmp",models_folder=models)

    tNameId = bt.Feature_id_transform(min_size=0,exclude_missing=True)
    tNameId.set_input_feature("name")
    tNameId.set_output_feature("nameId")
    svmTransform = bt.Svmlight_transform(included=["f1","f2","f3","f4"] )
    svmTransform.set_output_feature("svmfeatures")
    p.add(tNameId)
    p.add(svmTransform)
    p.fit_transform()

if __name__ == '__main__':
    parser = argparse.ArgumentParser(prog='bbm_pipeline')
    parser.add_argument('--events', help='events folder', required=True)
    parser.add_argument('--features', help='output features folder', required=True)
    parser.add_argument('--models', help='output models folder', required=True)

    args = parser.parse_args()
    opts = vars(args)

    run_pipeline([args.events],args.features,args.models)
{% endhighlight %}

The pipeline can be run as:

{% highlight bash %}
   make data/iris/features/1/features
{% endhighlight %}

This will call Docker to run:

{% highlight bash %}
docker run --rm -t -v ${PWD}/data:/data iris_pipeline bash -c "python /pipeline/iris_pipeline.py --events /data/iris/events/1 --features /data/iris/features/1 --models /data/iris/models/1"
{% endhighlight %}

The first few lines of the input events JSON look like:

{% highlight json %}
{"f1": 6.1, "f2": 3.0, "f3": 4.9, "f4": 1.8, "name": "Iris-virginica"}
{"f1": 5.0, "f2": 3.2, "f3": 1.2, "f4": 0.2, "name": "Iris-setosa"}
{"f1": 5.7, "f2": 2.9, "f3": 4.2, "f4": 1.3, "name": "Iris-versicolor"}
{"f1": 5.2, "f2": 2.7, "f3": 3.9, "f4": 1.4, "name": "Iris-versicolor"}
{% endhighlight %}

This is transformed by the pipeline into:

{% highlight json %}
{"f1": 6.1, "f2": 3.0, "f3": 4.9, "f4": 1.8, "name": "Iris-virginica", "nameId": 1, "svmfeatures": {"1": 6.1, "2": 3.0, "3": 4.9, "4": 1.8}}
{"f1": 5.0, "f2": 3.2, "f3": 1.2, "f4": 0.2, "name": "Iris-setosa", "nameId": 2, "svmfeatures": {"1": 5.0, "2": 3.2, "3": 1.2, "4": 0.2}}
{"f1": 5.7, "f2": 2.9, "f3": 4.2, "f4": 1.3, "name": "Iris-versicolor", "nameId": 3, "svmfeatures": {"1": 5.7, "2": 2.9, "3": 4.2, "4": 1.3}}
{"f1": 5.2, "f2": 2.7, "f3": 3.9, "f4": 1.4, "name": "Iris-versicolor", "nameId": 3, "svmfeatures": {"1": 5.2, "2": 2.7, "3": 3.9, "4": 1.4}}
{% endhighlight %}

## Vowpal Wabbit Model<a name="vw"></a>
We will create a Vowpal Wabbit model from our transformed features. We will use the [Seldon python package](python-package.html) which has a wrapper to allow easy creation of Vowpal wabbit models. 

{% highlight bash %}
make data/iris/vw/1
{% endhighlight %}

Which runs:

{% highlight bash %}
docker run --rm -t -v ${PWD}/data:/data seldonio/vw_train bash -c "cd /vw/vw_train ; python vw_train.py --client iris --day 1 --inputPath /data/ --outputPath /data/ --vwArgs '--passes 3 --oaa 3' --target nameId --include f1 f2 f3 f4 --train_filename train.vw --target_readable name"
{% endhighlight %}

For more details see [here](offline-prediction-models.html#vw).

## Create an XGBoost model<a name="xgboost"></a>
We will create an XGBoost model from our transformed features. We will use the [Seldon python package](python-package.html) which has a wrapper to allow easy creation of XGBoost models. 

{% highlight bash %}
make data/iris/vw/1
{% endhighlight %}

Which runs:

{% highlight bash %}
docker run --rm -t -v ${PWD}/data:/data seldonio/xgboost_train bash -c  "cd /xgboost/xgboost_train ; python xgboost_train.py --client iris --inputPath /data --outputPath /data --day 1 --target nameId --svmFeatures svmfeatures --target_readable name"
{% endhighlight %}

For more details see [here](offline-prediction-models.html#xgboost).

# Vowpal Wabbit Microservice<a name="vw-microservice"></a>
Now that we have built a VW model we can run a VW predictor as a microservice that will take in raw features, run our saved feature extraction pipeline and pass these features to VW to score returning a result.

This service can be started by running the command below which will start a microservice on port 5000.

{% highlight bash %}
make vw_runtime
{% endhighlight %}

Which runs:

{% highlight bash %}
docker run --name="vw_runtime" -d -v ${PWD}/data:/data -p 5000:5000 seldonio/vw_runtime bash -c "cd /vw/vw_runtime;python setup.py --client iris --day 1 --inputPath /data --exclude svmfeatures ;./start_vw_service.sh"
{% endhighlight %}

We can test this service by calling

{% highlight bash %}
make test_vw_runtime
{% endhighlight %}

Which uses curl to fire a JSON test set of feeatures to the microservice:

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

This shows the predition to be "Iris-setosa".


# XGBoost Microservice<a name="xgboost-microservice"></a>
Now that we have built an XGBoost model we can run an XGBoost predictor as a microservice that will take in raw features, run our saved feature extraction pipeline and pass these features to XGBoost to score returning a result.

This service can be started by running the command below which will start a microservice on port 5001.

{% highlight bash %}
make xgboost_runtime
{% endhighlight %}

Which runs:

{% highlight bash %}
docker run --name="xgboost_runtime" -d -p 5001:5000 -v ${PWD}/data:/data seldonio/xgboost_runtime bash -c "cd xgboost/xgboost_runtime ; python setup.py --client iris --day 1 --inputPath /data --svmFeatures svmfeatures ; ./start-service.sh"
{% endhighlight %}

We can test this service by calling

{% highlight bash %}
make test_xgboost_runtime
{% endhighlight %}

Which uses curl to fire a JSON test set of feeatures to the microservice:

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
				"host":"localhost",
				"port":3306,
				"user":"user1",
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
curl  -G "http://127.0.0.1:8080/seldon-server/js/predict?consumer_key=<CONSUMER_KEY_HERE>&jsonpCallback=callback" --data-urlencode 'json={"f1": 4.6, "f2": 3.2, "f3": 1.4, "f4": 0.2}'
{% endhighlight %}

This should give a response like:
{% highlight json %}
callback({"size":3,"requested":0,"list":[{"prediction":0.1820051759757194,"predictedClass":"Iris-virginica","confidence":1.0},{"prediction":0.5443882835483635,"predictedClass":"Iris-setosa","confidence":1.0},{"prediction":0.27360654047591704,"predictedClass":"Iris-versicolor","confidence":1.0}]})
{% endhighlight %}
