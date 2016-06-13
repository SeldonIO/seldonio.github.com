---
layout: default
title: Iago
---

# Load Test Seldon with Iago
[Iago])(https://github.com/twitter/iago) is a load generator created by Twitter. We provide a docker container and associated Kubernetes Pod and Service that can be used to load test Seldon configurations.

The steps to run a iago load test are described below. A detailed benchmarking guide for Seldon can be found [here](benchmark-guide.html)

## Create a replay script
Iago requires a replay script containing HTTP calls it can make to generate load. The folder ```seldon-server/docker/iago``` contains the Docker container for Iago. This folder also contains two shell scripts to create replay scripts for content recommendation and prediction endpoints.

 * You must have a running Seldon Kubernetes cluster to run these scripts as they use the seldon-cli to get the consumer keys for the clients.

For content recommendation use ```create_recommendation_replay.sh```. It you run it the arguments needed will be printed
{% highlight bash %}
./create_recommendation_replay.sh 
need <client name> <file> <num_items> <num_api_calls> 
{% endhighlight %}

 * <client name> : the name of the Seldon client to create a replay script for
 * <file> : the replay file to create
 * <num_items> : the number of items to retrieve from the db for generating random item API calls representing activity
 * <num_api_calls> : the number of API calls to create. The actual number of HTTP calls will be double this as 1 action call (e.g. page view) and 1 recommendation call will be created.

An example run of this script would be:
{% highlight bash %}
   ./create_recommendation_replay.sh ml10m /seldon-data/loadtests/example.ml10m.replay.txt  10000 10000 
{% endhighlight %}

To create a replay script for the prediction endpoint use ```create_prediction_replay.py```. An example call to this script for the iris example is provided in ```create_prediction_replay_iris.sh```. The python script allows you to specify each feature that can be used as input to a prediction call with a type and range of values. At present, this is limited to numeric features only. Random features in the range will be created for the replay script. The example ```create_prediction_replay_iris.sh``` is shown below:
{% highlight bash %}
#!/bin/bash

set -o nounset
set -o errexit

if [ "$#" -ne 3 ]; then
    echo "need <client name> <file> <num_api_calls>"
    exit -1
fi

CLIENT=$1
FILE=$2
NUM_API=$3

seldon-cli -q keys --client-name ${CLIENT} --scope js > key.json
python create_prediction_replay.py --key key.json --replay ${FILE}  --feature '{"name":"f1","type":"numeric","min":0,"max":5}' --feature '{"name":"f2","type":"numeric","min":0,"max":5}' --feature '{"name":"f3","type":"numeric","min":0,"max":5}' --feature '{"name":"f4","type":"numeric","min":0,"max":5}' --num ${NUM_API}
{% endhighlight %}


## Start Iago Load Test
To start iago [create a Seldon Kubernetes cluster](install.html) and go to the folder ```seldon-server/kubernetes/conf/dev``` and run:
{% highlight bash %}
   cd seldon-server/kubernetes/conf/dev
   kubectl create -f iago.json
{% endhighlight %}

Upload the replay script created above to the Seldon presistent data store exposed as /seldon-data inside the container. 

Log into iago container:
{% highlight bash %}
   kubectl exec -ti `kubectl get pods | grep iago | cut -d' ' -f1` -- /bin/bash
{% endhighlight %}

Start a load test using the uploaded replay script at a given number of API requests per second. For example, for a replay script in ```/seldon-data/loadtest``` which is to be run at 100 requests per second:

{% highlight bash %}
  ./start-load-test.sh /seldon-data/loadtest/example.ml10m.replay.txt 100
{% endhighlight %}

## Get analytics
Iago provide an endpoint that provides analytics on port 9994 of the exposed service. For example if you specified a loadbalancer in your config then the latency endpoint would be at:

{% highlight bash %}
http://<iago-ui-loadbalncer>:9994/graph/?g=metric:client/request_latency_ms
{% endhighlight %}


For a full list of metrics see the [Iago documentation](https://github.com/twitter/iago#metrics).