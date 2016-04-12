---
layout: default
title: Prediction Example
---

# Creating a Prediction Service

This example will take you through creating a simple prediction service to serve predictions for the [Iris dataset](http://archive.ics.uci.edu/ml/datasets/Iris). The predictive service will be deployed as a microservice inside Kubernetes allowing roll-back/up for scalable maintenance. It will be accessed via the Seldon server allowing you to run multiple algorithms and manage predictive services for many clients.

 * [Prerequisites](#prerequisites)
 * [Create Model](#model)
 * [Start Microservice](#microservice)
 * [Serve Predictions](#predictions)
 * [Next Steps](#next-steps)

# Prerequisites<a name="prerequisites"></a>

 * [You have installed Seldon on a Kubernetes cluster](install.html)
 * You haved added ```seldon-server/kubernetes/bin``` to you shell PATH environment variable.

# Create a Predictive Model<a name="model"></a>
The first step is to create a predictive model based on some training data. In this case we have prepacked into 3 Docker images the process of creating a model for the Iris dataset using three popular machine learning toolkits [XGBoost](https://github.com/dmlc/xgboost), [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki) and [Keras](http://keras.io/). Wrappers to call these libraries have been added to our python library to make integrating them as a microservice easy. However, you can build your model using any machine learning library. 

The three images are:

 * seldonio/iris_xgboost : an XGBoost model for the iris dataset
 * seldonio/iris_vw : a VW model for the iris dataset
 * seldonio/iris_keras : a keras model for the iris dataset

For details on building models using our python library see [here](prediction-pipeline.html).

# Start a microservice<a name="microservice"></a>

At runtime Seldon requires you expose your model scoring engine as a microservice API. In this example case the same image to create the models also exposes it for runtime scoring when run. We can start our chosen microservice using ```kubernetes/bin/run_prediction_microservice.sh```. This script takes 4 arguments

  * A name for the microservice
  * An image to pull that can be run to start the microservice
  * A version for the image
  * A client to connect the microservice to

The script create a Kubernetes deployment for the microservice in ```kubernetes/conf/microservices```. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.

For example to start the XGBoost Iris microservice on the client  "test" (created by seldon-up.sh on startup):

{% highlight bash %}
run_prediction_microservice.sh iris-xgboost-example seldonio/iris_xgboost 1.0 test
{% endhighlight %}

The script will use the seldon-cli to update the "test" client to add the microservice as a runtime algorithm. 

# Serve Predictions<a name="predictions"></a>

You can now call the seldon server using the seldon CLI to test:

{% highlight bash %}
seldon-cli --quiet api --client-name test --endpoint /js/predict --json '{"f1":1,"f2":2.7,"f3":5.3,"f4":1.9}'
{% endhighlight %}

The respone should be like:
{% highlight json %}
{"size":3,"requested":0,"list":[{"prediction":0.00252304,"predictedClass":"Iris-setosa","confidence":0.00252304},{"prediction":0.00350009,"predictedClass":"Iris-versicolor","confidence":0.00350009},{"prediction":0.993977,"predictedClass":"Iris-virginica","confidence":0.993977}]}
{% endhighlight %}

# Next Steps<a name="next-steps"></a>

 * [Detailed Guide for General prediction](prediction-guide.html)

