---
layout: default
title: Prediction Deploy
---

# General Predictive Services
This gudie takes you through the steps to serve general predictive models through Seldon.

 * [Overview](#overview)
 * [Create a client](#client)
 * [Collect data](#data)
 * [Create a prediction model](#model)
 * [Serve predictions](#predict)
 * [Advanced Settings](#advanced)

# **Overview**<a name="overview"></a>

Seldon provides the ability to build supervised learning based predictive models and put them into production at scale. The strutcture of a general predictive pipeline is show in the diagram below:

![Predictive Data Pipelines](/img/predictive-data-pipelines.png)

There are offline and realtime components. Raw data to be used for creating a predictive pipeline is sent to Seldon via its REST API in real time. This data can be sent as arbitrary JSON which allows complete freedom for a client to provide whatever data is available. It is normally the case that this raw JSON data sent to Seldon is not in the best format to directly build machine learning models. Therefore, in the offline modelling stage the data is first sent through an (optional) set of feature transformations to extract and create appropriate features that are useful for creating predictive models. After these transformations a model can be built to predict some target feature in the data based on the extracted/transformed features. 

At runtime as prediction calls come in the same set of transformations performed offline will need to be repeated to create the same set of final features to test against the model. Once the transformations have been done the features can be scored and a predictive result returned to the client in real time.


# **Create a Client**<a name="client"></a>
Setup a client in Seldon with [```seldon-cli client```](seldon-cli.html#client). This will create consumer keys that can be used in the next section.

# **Collect Data**<a name="data"></a>
Events can be sent to Seldon via the [prediction API](api-prediction.html). They will be transfered from the Seldon Server(s) to a central store using Fluentd and stored at ```/seldon-data/logs/events.<year>/<month>/<day/<hour>/file.gz``` as JSON if the default Fluentd configuration is kept. These files contain events for all the clients setup as many clients can use the same API via different JS and Oauth consumer keys. To separate out the events for each client into individual files we provide a spark job that processes these files and separates them into separate folders for processing by modelling jobs, usinf [```seldon-cli client --action processevents```](/seldon-cli.html#client).

You have other custom options for integration if needed:

  * You can also place any historical data you may have into ```/seldon-data/seldon-models/<client>/events/<year>/<month>/<day>``` as JSON.
  * You can change the Fluentd configuration to push the data to a custom location
  * You can bypass this section an use your own events datastore to build predictive models and serve them through Seldon

# **Create a prediction model**<a name="model"></a>

You can create your machine learning model using the toolkit of your choice. However, to make building predictive pipelines easier and to allow them to be used at runtime as well as during modeling we presently provide a python library that allows you to create pandas and scikit-learn compatable predictive pipelines. For details on using these see [here](prediction-pipeline.html).

# **Serve Predictions**<a name="predict"></a>
To serve predictions for your model you need to provide a runtime microservice that conforms to the [microserice prediction API](/api-prediction.html) packaged as a Docker container. If your model is built using our python predictive pipelines you can easily package it as a [microservice](prediction-pipeline.html#microservice).

Once packaged as a Docker container the microservice can be started using the command line script [start-microservice](scripts.html/#start-microservice).

The script creates a Kubernetes deployment for the microservice in ```kubernetes/conf/microservices```. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.

For example to start the XGBoost Iris microservice on the client  "test":

{% highlight bash %}
start-microservice --type prediction --client test -i iris-xgboost seldonio/iris_xgboost:2.1 rest 1.0
{% endhighlight %}

The script will use the seldon-cli to update the "test" client to add the microservice as a runtime algorithm. You can now get predictions via the Seldon API.

# **Advanced Settings**<a name="advanced"></a>

##  Run A/B Tests
To test differenet prediction algorithms in a live setting you will want to run A/B tests.

If you have two microservices you want to test in an A/B test you can use the script [start-microservice](scripts.html/#start-microservice).

An example to start two variantions of the example Iris predictors using Xgboost and Scikit-learn variants and sending 50% of the traffic to each is shown below:

{% highlight bash %}
start-microservice --type prediction --client test -i iris-xgboost seldonio/iris_xgboost:2.1 rest 0.5 -i iris-scikit seldonio/iris_scikit:2.1 rest 0.5
{% endhighlight %}

The script will start both microservices and provide the correct configuration via the seldon CLI. The configuration used for the above example is shown below:

{% highlight json %}
{
    "variations": [
        {
            "config": {
                "algorithms": [
                    {
                        "config": [
                            {
                                "name": "io.seldon.algorithm.external.url",
                                "value": "http://iris-xgboost:5000/predict"
                            },
                            {
                                "name": "io.seldon.algorithm.external.name",
                                "value": "iris-xgboost"
                            }
                        ],
                        "name": "externalPredictionServer"
                    }
                ]
            },
            "label": "iris-xgboost",
            "ratio": 0.5
        },
        {
            "config": {
                "algorithms": [
                    {
                        "config": [
                            {
                                "name": "io.seldon.algorithm.external.url",
                                "value": "http://iris-scikit:5000/predict"
                            },
                            {
                                "name": "io.seldon.algorithm.external.name",
                                "value": "iris-scikit"
                            }
                        ],
                        "name": "externalPredictionServer"
                    }
                ]
            },
            "label": "iris-scikit",
            "ratio": 0.5
        }
    ]
}
{% endhighlight %}

See [here](seldon-cli.html#predict_alg) on how to use the Seldon CLI to set a custom prediction algorithm configuration.
