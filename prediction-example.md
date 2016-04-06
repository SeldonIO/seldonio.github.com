---
layout: default
title: Prediction Example
---

# Creating a Prediction Service

This example will take you through creating a simple example prediction service to serve predictions for the Iris dataset. The predictive service will be deployed as a microservice inside Kubernetes allowing roll-back/up for scalable maintenance. It will be accessed via the Seldon server allowing you to run multiple algorithms and manage predictive services for many clients.

 * [Create Model](#model)
 * [Start Microservice](#microservice)
 * [Serve Predictions](#predictions)

# Create a Predictive Model<a name="model"></a>
The first step is to create a predictive model based on some training data. In this case we have prepacked into 3 Docker images the process of creating a model for the Iris dataset using three popular machine learning toolkits XGBoost, Vowpal Wabbit and Keraa. Wrappers to call these libraries have been added to our python library to make integrating them as a microservice easier. However, you can build your model using any machine learning library. 

The three images are:

 * seldonio/iris_xgboost : an XGBoost model for the iris dataset
 * seldonio/iris_vw : a VW model for the iris dataset
 * seldonio/iris_keras : a keras model for the iris dataset

For details on building models using our python library see here.

# Start a microservice<a name="microservice"></a>

At runtime Seldon required you expose your model scoring engine as a microservice API. In this example case the same image to create the models also exposes the it for runtime scoring when run. We can start our chosen microservice using ```kubernetes/scripts/run_prediction_microservice.sh```. This script takes 3 arguments

  * A name for the microservice
  * An image to pull that can be run to start the microservice
  * A version for the image

The script create a Kubernetes deployment for the microservice in ```. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.

For example to start the XGBoost Iris microservice on the client  "test" (created by seldon-up.sh on startup):

{% highlight bash %}
kubernetes/scripts/run_prediction_microservice.sh iris-xgboost-example seldonio/iris_xgboost 1.0 test
{% endhighlight %}

The script will use the seldon-cli to update the "test" client to add the microservice as a runtime algorithm. 

# Serve Predictions<a name="predictions"></a>

You can now call the seldon server using the client details for the test client to get predictions. We have provided an example script to do this in ```kubernetes/examples/iris/get_iris_prediction.sh```. Running this should produce an output like:


{% highlight json %}
{
  "size": 3,
  "requested": 0,
  "list": [
    {
      "prediction": 0.00252304,
      "predictedClass": "Iris-setosa",
      "confidence": 0.00252304
    },
    {
      "prediction": 0.00350009,
      "predictedClass": "Iris-versicolor",
      "confidence": 0.00350009
    },
    {
      "prediction": 0.993977,
      "predictedClass": "Iris-virginica",
      "confidence": 0.993977
    }
  ]
}
{% endhighlight %}


