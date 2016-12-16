---
layout: default
title: Benchmarking prediction services
---

# Guide to Benchmark Prediction Services
This guide will go through detailed steps to show how a Seldon prediction service can be benchmarked and will compare REST and gRPC variants. The service benchmarked will be the [MNIST digit classifider demo](tensorflow-deep-mnist-example-docker.html). We will use the [locust](http://locust.io/) load testing framework which is easier to extend to create gRPC tests than [Iago](https://github.com/twitter/iago).

The steps taken will be

 * Launch a 3 node cluster on AWS
 * Create Seldon configuration and Launch onto cluster
 * For REST and gRPC variants: 
     * Start the MNIST demo (in REST or gRPC variant)
     * Launch a locust load test

## Create Kubernetes cluster on AWS

For our test we used AWS but other cloud providers can be used. You will need persistent storage to run Seldon. For this test we used GlusterFS. 

 * [Create a Kubernetes cluster on AWS](http://kubernetes.io/docs/). 
   * We created one with 3 m3.large minions and a m3.large master.
 * [Create a GlusterFS cluster in an AWS VPC](glusterfs.html).

## Launch Seldon

 * Update Seldon Kubernetes configuration
    * Update ```seldon-server/kubernetes/conf/Makefile```, setting the glusterfs ip addresses as appropriate for your glusterfs configuration. Also, change the SELDON_SERVICE_TYPE to LoadBalancer. For example:
{% highlight bash %}
   DATA_VOLUME="glusterfs": {"endpoints": "glusterfs-cluster","path": "gv0","readOnly": false}
   SELDON_SERVICE_TYPE=LoadBalancer
   GLUSTERFS_IP1=192.168.0.149
   GLUSTERFS_IP2=192.168.0.248
{% endhighlight %}
   Create the conf with ```make clean conf```

 * Label one node as a locust loadtest node. We will use this node to run the loadtester later. Your node name will differ.
{% highlight bash %}
   kubectl label nodes ip-172-20-0-71.eu-west-1.compute.internal role=locust
{% endhighlight %}
 * Cordon off the locust loadtest node so its not used for Seldon. 
{% highlight bash %}
   kubectl cordon ip-172-20-0-71.eu-west-1.compute.internal
{% endhighlight %}
 * Start Seldon, with glusterfs and without Spark (not needed for this test)
{% highlight bash %}
   SELDON_WITH_GLUSTERFS=true SELDON_WITH_SPARK=false seldon-up
{% endhighlight %}

## Download pretrained MNIST model
We follow the steps outlined in the [MNIST digit classifier demo](tensorflow-deep-mnist-example-docker.html) to setup a client and download a model for that client.

 * Create a Seldon client for the predictions
{% highlight bash %}
seldon-cli client --action setup --db-name ClientDB --client-name deep_mnist_client
{% endhighlight %}
 * Download a pretrained deep neural network model for digit recognition
{% highlight bash %}
cd kubernetes/conf/examples/tensorflow_deep_mnist
kubectl create -f load-model-tensorflow-deep-mnist.json
{% endhighlight %}

Wait until the model has finished downloading by checking the job with ```kubetctl get jobs```


## Start a REST MNIST prediction microservice

{% highlight bash %}
start-microservice --type prediction --client deep_mnist_client -p tensorflow-deep-mnist /seldon-data/seldon-models/tensorflow_deep_mnist/1/ rest 1.0
{% endhighlight %}


## Run REST load test

  * Uncordon the locust node so we can start the locust loadtest services in it.
{% highlight bash %}
   kubectl uncordon ip-172-20-0-71.eu-west-1.compute.internal
{% endhighlight %}
  * Launch a REST based load test sending random data to the server
{% highlight bash %}
  launch-locust-load-test --seldon-client deep_mnist_client --test-type js-predict
{% endhighlight %}

## REST Results

![locust REST results](/img/locust_rest.png)


## Clean up the load test resources

 * Stop the REST microservice. A JSON kubernetes config would have been created for the deployment and this can be used to clean up the resources:
{% highlight bash %}
   cd kubernetes/conf/microservices
   kubectl delete -f microservice-tensorflow-deep-mnist.json 
{% endhighlight %}

 * Stop the locust load test pods. JSON files would have been created which can be used to clean up the resources:
{% highlight bash %}
   cd kubernetes/conf/dev
   kubectl delete -f locust-master.json
   kubectl delete -f locust-slave.json
{% endhighlight %}

 * Recordon the load test node
{% highlight bash %}
   kubectl cordon ip-172-20-0-71.eu-west-1.compute.internal
{% endhighlight %}

## Start the gRPC MNIST prediction microservice

{% highlight bash %}
start-microservice --type prediction --client deep_mnist_client -p tensorflow-deep-mnist /seldon-data/seldon-models/tensorflow_deep_mnist/1/ rpc 1.0
{% endhighlight %}

## Run gRPC load test

  * Uncordon the locust node so we can start the locust loadtest service in it.
{% highlight bash %}
   kubectl uncordon ip-172-20-0-71.eu-west-1.compute.internal
{% endhighlight %}
  * Launch a gRPC based load test sending random data to the server
{% highlight bash %}
  launch-locust-load-test --seldon-client deep_mnist_client --test-type grpc-predict
{% endhighlight %}

## gRPC Results

![locust gRPC results](/img/locust_grpc.png)

# Discussion
On the average response time for REST is over 50% slower than for gRPC.

The percentiles from the two tests are as follows.

{% highlight bash %}
"Name",			"# requests",	"50%",	"66%",	"75%",	"80%",	"90%",	"95%",	"98%",	"99%",	"100%"
"grpc http://seldon-server:80" ,43521	14	16	18	19	25	36	67	110	5045
"GET /js/predict"	       ,42462	20	25	30	36	63	110	190	240	717
{% endhighlight %}

The advantage of gRPC increases as the percentiles increase except for the 100% percentile which suggests there was some delay at the start or outlier that needs further investigation.

The locust tests were done with 50 clients, each waiting between 900 and 1000ms between calls. Locust unlike [Iago](https://github.com/twitter/iago) does not allow you to create a fixed request rate so some delays in the load testing framework itself between REST and gRPC implementations could effect the response times. However, we found it less easy to develop a gRPC test in Iago.

