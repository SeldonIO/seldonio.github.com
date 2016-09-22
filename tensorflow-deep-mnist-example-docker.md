---
layout: default
title: TensorFlow Deep MNIST Demo
---

# TensorFlow Deep MNIST Demo

This example will take you through creating a microservice that recognizes numbers between 0 and 9, based on the CNN model from the [tensorflow deep MNIST demo](https://www.tensorflow.org/versions/r0.10/tutorials/mnist/pros/index.html). In this example you will learn how to deploy the microservice from the prepackage docker image available in seldon-server. 

If you want to learn how to build this image from your model, check out our [TensorFlow Digit Classifier Advanced Tutorial](tensorflow-deep-mnist-example.html).

<p style="margin:auto; width:50%;">
<iframe width="560" height="315" src="https://www.youtube.com/embed/T4Y5mS75z9I" frameborder="0" allowfullscreen></iframe>
</p>

## Prerequisites

 * You have [installed Seldon](install.html) on a Kubernetes cluster.

## Create a Predictive Model

In this demo, we have created and pre-packaged a predictive model built with [tensorflow](https://www.tensorflow.org/). A wrapper to call this library has been added to our python library to make integrating it as a microservice easy. However you can build your own model using any machine learning library.

The image is seldonio/tensorflow_deep_mnist.

For details on building the model see the [tensorflow deep mnist tutorial](https://www.tensorflow.org/versions/r0.10/tutorials/mnist/pros/index.html).

For details on building a docker image from your model see our [detailed deep mnist tutorial](tensorflow-deep-mnist-example.html).


## Train the model

The model training can be accomplished by running the Kubernetes job in ```kubernetes/conf/examples/tensorflow_deep_mnist/train-tensorflow-deep-mnist.json``` which should have been created when you followed the [install and configuration steps](install.html).

Create the kubernetes job to train the model:

{% highlight bash %}
cd kubernetes/conf/examples/tensorflow_deep_mnist
kubectl create -f train-tensorflow-deep-mnist.json
{% endhighlight %}

This job will:
 * Run the docker image containing the tensorflow model in your cluster;
 * Train the convolutional neural network on the MNIST dataset; 
 * Save the predictive pipeline to persistent storage;
 * Kill the docker container.

This job can take several hours to complete on a machine without a CUDA enabled GPU. If you do not want to wait this long you can load a pretrained model using the Kubernetes job in ```kubernetes/conf/examples/tensorflow_deep_mnist/load-model-tensorflow-deep-mnist.json```.

Create the kubernetes job to load the model:

{% highlight bash %}
cd kubernetes/conf/examples/tensorflow_deep_mnist
kubectl create -f load-model-tensorflow-deep-mnist.json
{% endhighlight %}

## Create Client

You will need to create a Seldon Client to serve as an entry point for your microservices. You can create a client in one line using the seldon cli as below. For more details on how to use the seldon cli please refer to our [seldon-cli documentation](seldon-cli.html).

{% highlight bash %}
seldon-cli client --action setup --db-name ClientDB --client-name deep_mnist_client
{% endhighlight %}

Here we setup a new client called deep_mnist_client that uses the database ClientDB (created when you installed seldon on your cluster).

## Serve Predictions

To serve predictions we will load the saved pipeline into a microservice. This can be accomplished by using the script ```run_prediction_pipeline_microservice.sh``` in ```seldon-server/kubernetes/bin```.

{% highlight bash %}
run_prediction_pipeline_microservice.sh tensorflow-deep-mnist /seldon-data/seldon-models/tensorflow_deep_mnist/1/ deep_mnist_client 1
{% endhighlight %}

This will load the pipeline saved in ```/seldon-data/seldon-models/tensorflow_deep_mnist/1/``` and create a single replica microservice called tensorflow-deep-mnist. It will activate this for the "deep-mnist-service" created in the previous step.

## Testing your service

The microservice takes as input a vector of 784 floats corresponding to the pixels of a 28x28 image and returns a list of probabilities for each number between 0 and 9. In order to test it you can use the [flask webapp](tensorflow-deep-mnist-webapp.html) we have created for this purpose.


