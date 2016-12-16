---
layout: default
title: command line scripts
---

# Command Line Scripts
Seldon provides several scripts to aid starting Seldon, provisioning services and stopping Seldon.

* [**seldon-up**](#seldon-up)
* [**seldon-down**](#seldon-down)
* [**seldon-cli**](#seldon-cli)
* [**start-microservice**](#start-microservice)
* [**launch-locust-load-test**](#launch-locust-loadtest)

# <a name="seldon-up"></a>**seldon-up**

## Synopsis
Create Seldon on a running kubernetes cluster.

{% highlight bash %}
seldon-up
{% endhighlight %}

## Examples
To launch seldon with all components run
{% highlight bash %}
seldon-up
{% endhighlight %}

To start with GlusterFS run 
{% highlight bash %}
SELDON_WITH_GLUSTERFS=true seldon-up
{% endhighlight %}

To start without a Spark cluster 
{% highlight bash %}
SELDON_WITH_SPARK=false seldon-up
{% endhighlight %}

# <a name="seldon-down"></a>**seldon-down**

## Synopsis
Shutdown Seldon running on a Kubernetes cluster

{% highlight bash %}
seldon-down
{% endhighlight %}

# <a name="seldon-cli"></a>**seldon-cli**
See the detailed [seldon-cli](seldon-cli.html) documentation.

# <a name="start-microservice"></a>**start-microservice**

## Synopsis
Start one or more  microservices for a particualar client. The microservices can be REST or for predictions also gRPC based.
The script allows you to start microservices of two types:

 * Microservices already package as a Docker image
 * Microservices started from a saved python scikit-learn pipeline using [pyseldon](prediction-pipeline.html)

{% highlight bash %}
usage: create_replay [-h] [-i name image microservice API ratio]
                     [-p name folder microservice API ratio] --client CLIENT
                     [--replicas REPLICAS] --type {recommendation,prediction}

optional arguments:
  -h, --help            show this help message and exit
  -i name image microservice API ratio
                        microservice image defn: <name> <image> <API type
                        (rest or rpc)> <ratio>
  -p name folder microservice API ratio
                        microservice from pipeline defn: <name> <folder> <API
                        type (rest or rpc) <ratio>
  --client CLIENT       client name
  --replicas REPLICAS   number of replicas
  --type {recommendation,prediction}
                        microservice type
{% endhighlight %}

## Examples

Start a recommendation microserice from a built Docker image exposed as  REST endpoint for the client "reuters". See the worked [Reuters content recommendation example](content-recommendation-example.html).
{% highlight bash %}
start-microservice --type recommendation --client reuters -i reuters-example seldonio/reuters-example:2.0.7 rest 1.0
{% endhighlight %}

Start a prediction REST microservice from a saved pipeline previoiusly saved to /seldon-data/seldon-models/finefoods/1 for client "test". See the worked [sentiment analysis demo](sentiment-demo.html).
{% highlight bash %}
start-microservice --type prediction --client test -p finefoods-xgboost /seldon-data/seldon-models/finefoods/1/ rest 1.0
{% endhighlight %}

Start a prediction microservice from an xgboost mode exposed as a REST service and packaged in  docker image. See worked example in the [Iris prediction demo](prediction-example.html).
{% highlight bash %}
start-microservice --type prediction --client test -i iris-xgboost seldonio/iris_xgboost:2.1 rest 1.0
{% endhighlight %}

Start and AB test with two microservices.
{% highlight bash %}
start-microservice --type prediction --client test -i iris-xgboost seldonio/iris_xgboost:2.1 rest 0.5 -i iris-scikit seldonio/iris_scikit:2.1 rest 0.5
{% endhighlight %}

Start gRPC microservice for Iris demo
{% highlight bash %}
start-microservice --type prediction --client test -i xgboostrpc seldonio/iris_xgboost_rpc:2.1 rpc 1.0
{% endhighlight %}


# <a name="launch-locust-loadtest"></a>**launch-locust-loadtest**

## Synopsis
Create a locust loadtest. Presently for prediction services only. Handles REST and gRPC

{% highlight bash %}
usage: launch-locust-load-test [-h] --seldon-client SELDON_CLIENT
                               [--locust-slaves LOCUST_SLAVES]
                               [--locust-hatch-rate LOCUST_HATCH_RATE]
                               [--locust-clients LOCUST_CLIENTS] 
			       --test-type {js-predict,rpc-predict}
                               [--seldon-grpc-endpoint SELDON_GRPC_ENDPOINT]
                               [--seldon-oauth-endpoint SELDON_OAUTH_ENDPOINT]
                               [--seldon-predict-default-data-size SELDON_PREDICT_DEFAULT_DATA_SIZE]

optional arguments:
  -h, --help            show this help message and exit
  --seldon-client SELDON_CLIENT
                        client name
  --locust-slaves LOCUST_SLAVES
                        number of slaves to run
  --locust-hatch-rate LOCUST_HATCH_RATE
                        locust hatch rate
  --locust-clients LOCUST_CLIENTS
                        number of locust clients
  --test-type {js-predict,grpc-predict}
                        type of test to run
  --seldon-grpc-endpoint SELDON_GRPC_ENDPOINT
                        seldon grpc endpoint
  --seldon-oauth-endpoint SELDON_OAUTH_ENDPOINT
                        seldon oauth endpoint
  --seldon-predict-default-data-size SELDON_PREDICT_DEFAULT_DATA_SIZE
                        the size of the default list of random floats to send
                        to predict endpoint

{% endhighlight %}

## Examples

{% highlight bash %}
# launch grpc prediction load test
launch-locust-load-test --seldon-client deep_mnist_client --test-type grpc-predict
{% endhighlight %}



