---
layout: default
title: INNe anomaly detection  Demo
---

# INNe anomaly detection Demo

This example will take you through creating a microservice that detect anomalies using [iNNE](inneurl) technique. In this example you will learn how to deploy the microservice from the prepackage docker image available in seldon-server. 

If you are interested on theory behind iNNE technique, find out more about [iNNE anomaly detection](inne_anomaly_detector.html).


## Prerequisites

 * You have [installed Seldon](install.html) on a Kubernetes cluster.


## Train the model

The iNNE anomaly detector is trained on a dataset of generated samples. We have prepacked a docker image that generate the train dataset, fit the model and save the pipeline. To train the detector, we generate data by sampling 1000 random 4-dimesional vectors from 3 gaussian distributions centered at 2.0, 4.0 and 6.0 and with standard deviation of 0.1. Moreover we include 5 anomaly points by sampling 5 random 4-dimensional vectors from a gaussian centered at 0.0 and with standard deviation 0.1. This procedure results on a train dataset of 3005 samples.

## Create Microservice

At runtime Seldon requires you expose your model scoring engine as a microservice API. In this example case the same image to create the models also exposes it for runtime scoring when run. We can start our chosen microservice using the command line script start-microservice

The script creates a Kubernetes deployment for the microservice in kubernetes/conf/microservices. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.

To start the iNNE detector microservice on the client “test” (created by seldon-up on startup):

{% highlight bash %}
start-microservice --type prediction --client test -i inne seldonio/simulation_inne:1.0 rest 1.0
{% endhighlight %}

This will load the pipeline saved in ```/seldon-data/seldon-models/inne/1/``` and create a single replica REST microservice called inne. It will activate this for the "inne-client" created in the previous step.

## Serve anomaly detection

To obtain the anomaly score for any 4-dimesional vector

{% highlight bash %}
seldon-cli --quiet api --client-name test --endpoint /js/predict --json '{"data":{"f1":2.1,"f2":2.2,"f3":1.9,"f4":2.0}}'
{% endhighlight %}

The response should be 

{% highlight json %}
{
  "meta": {
    "puid": "206b92133954683a4b92236b7e15667579242cf5",
    "modelName": "model_inne",
    "variation": "inne"
  },
  "predictions": [
    {
      "prediction": 0.213911490796,
      "predictedClass": "Anomaly_score",
      "confidence": 0.213911490796
    },
    {
      "prediction": 0.786088509204,
      "predictedClass": "Complementary_score",
      "confidence": 0.786088509204
    }
  ],
  "custom": null
}

{% endhighlight %}

Since the anomaly detector uses the seldon prediction format for the response, the above data should be interpreted as follow:

*  if "PredictedClass" : "Anomaly_score", the keys "prediction" and "confidence" store the anomaly score for the vector, where 0 is not anomalous and 1 is the maximally anomalous 
*  if "PredictedClass" : "Complementary_score", the keys "prediction" and "confidence" store the non-anomaly score, which is 1 - anomaly_score

Since many samples (about one third) in the dataset are drawn from a gaussian centered in 2.0, the anomaly score for the sample is low. A vector such as '{"data":{"f1":0.1,"f2":0.2,"f3":0.9,"f4":0.0}}' should get an high anomaly score