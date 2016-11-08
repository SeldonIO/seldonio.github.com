---
layout: default
title: gRPC
---

# RPC based services
Seldon provides an RPC (Remote Procedure Call) interface using [gRPC](http://www.grpc.io/) with [protocol buffers](https://developers.google.com/protocol-buffers/). The following docs require a knowkedge of gRPC. 

# Seldon gRPC
The proto file for the Seldon gRPC services is shown below. It contains a single service Classifier for classification prediction calls that takes a Classificationrequest and returns a Classification Reply.

A *ClassificationRequest* has two parts
  
 * *meta* : metadata associated with the request, presently an optional prediction-id the client wishes to associate with the request.
 * *data* : A custom entry to hold the features needs for the prediction request defined by the user. 

A *ClassificationReply* has three parts

 * *meta* : metadata associated with the prediction
   * *puid* : prediction unique id either supplied by client in request or created by Seldon
   * *modelName* : the name of the model that satisfied the request
   * *variation* : the AB test variation used in satisfying the request (will be "default" if a single variation).
 * *predictions* : the predictions for each class
 * *custom* : optional custom additonal data that is defined the user

{% highlight proto %}
syntax = "proto3";

import "google/protobuf/any.proto";

option java_multiple_files = true;
option java_package = "io.seldon.api.rpc";
option java_outer_classname = "PredictionAPI";

package io.seldon.api.rpc;

service Classifier {
  rpc Predict (ClassificationRequest) returns (ClassificationReply) {}
}

//Classification Request

message ClassificationRequest {
 ClassificationRequestMeta meta = 1;
 google.protobuf.Any data = 2;
}

message ClassificationRequestMeta {
  string puid = 1;
}

// Classification Reply

message ClassificationReply {
 ClassificationReplyMeta meta = 1;
 repeated ClassificationResult predictions = 2;
 google.protobuf.Any custom = 3;
}

message ClassificationReplyMeta {
  string puid = 1;
  string modelName = 2;
  string variation = 3;
}

message ClassificationResult {
  double prediction  = 1;
  string predictedClass = 2;
  double confidence = 3;
}
{% endhighlight %}


# Deploying a gRPC Prediction Service 
The stages to deploy a gRPC service are shown below.

 1. Create custom proto buffer file
 1. Build model and package microservice using gRPC
 1. Inform Seldon of Java custom protocol buffers
 1. Launch gRPC microservice
 1. Test via REST or gRPC clients

We will work through these stages using the simple [Iris prediction dataset](http://archive.ics.uci.edu/ml/datasets/Iris) as an example. The code can be found at [docker/examples/iris/xgboost_rpc](https://github.com/SeldonIO/seldon-server/tree/master/docker/examples/iris/xgboost_rpc).

## Create Custom Proto Buffer
To hold the features required for a prediction request we will create the following proto buffer file:
{% highlight proto %}
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.seldon.microservice.iris";
option java_outer_classname = "IrisClassifier";

package io.seldon.microservice.iris;

message IrisPredictRequest {
  float f1 = 1;
  float f2 = 2;
  float f3 = 3;
  float f4 = 4;
}
{% endhighlight %}

## Build model and package as microservice
To build the model we will use XGBoost and its wrapper provided in the pyseldon python library. The model is the same as shown in the [Iris demo](prediction-example.html) so won't be discussed further here. We create python implementations of our protocol buffer and create a microservice wrapper to our built python pipeline model using the Microservice package as shown below. For this we need to create a custom data handler which will be call to extract a Pandas Dataframe from the RPC call data. The data will contain out custom IrisPredictRequest class which we need to unpack and translate into a Dataframe. The created Dataframe must match what is expected as input to your pipeline model.

{% highlight python %}
from concurrent import futures
import time
import sys, getopt, argparse
import seldon.pipeline.util as sutl
import random
import iris_pb2
import grpc
import google.protobuf
from google.protobuf import any_pb2
import pandas as pd 
from seldon.microservice.rpc import CustomDataHandler
from seldon.microservice import Microservices

class BadDataError(Exception):
    def __init__(self, value):
        self.value = value
    def __str__(self):
        return repr(self.value)

class IrisCustomDataHandler(CustomDataHandler):

    def getData(self, request):
        anyMsg = request.data
        dc = iris_pb2.IrisPredictRequest()
        success = anyMsg.Unpack(dc)
        if success:
            df = pd.DataFrame([{"f1":dc.f1,"f2":dc.f2,"f3":dc.f3,"f4":dc.f4}])
            return df
        else:
            context.set_code(grpc.StatusCode.INTERNAL)
            context.set_details('Invalid data')
            raise BadDataError('Invalid data')


if __name__ == "__main__":
    import logging
    logger = logging.getLogger()
    logging.basicConfig(format='%(asctime)s : %(levelname)s : %(name)s : %(message)s', level=logging.DEBUG)
    logger.setLevel(logging.INFO)


    parser = argparse.ArgumentParser(prog='microservice')
    parser.add_argument('--model-name', help='name of model', required=True)
    parser.add_argument('--pipeline', help='location of prediction pipeline', required=True)
    parser.add_argument('--aws-key', help='aws key', required=False)
    parser.add_argument('--aws-secret', help='aws secret', required=False)

    args = parser.parse_args()
    opts = vars(args)

    m = Microservices(aws_key=args.aws_key,aws_secret=args.aws_secret)
    cd = IrisCustomDataHandler()
    m.create_prediction_rpc_microservice(args.pipeline,args.model_name,cd)
{% endhighlight %}

The above model and resulting microservice has been packaged as a Docker container seldonio/iris_xgboost_rpc.

## Inform Seldon of Java custom protocol buffers
The front end server for Seldon is a Java server. In order to process the gRPC calls for a client prediction endpoint correctly it needs to know the custom request and optional reply proto buffer implementations. To do this you need to do two things:
 
 1. Create Java versions of your proto buffers
 1. Tell Seldon of a jar file containing the Java classes and the names of those classes

For the first step you can follow the protocol buffer docs and package the result. We provide a simple script to do this for you [create-proto-jar](scripts.html#create-proto-jar)

Assuming you have placed the iris.proto above on Seldon shared volume at /seldon-data/rpc/proto/iris.proto and want to output the jar at /seldon-data/rpc/jar/iris.jar you can run:

{% highlight bash %}
create-proto-jar.sh /seldon-data/rpc/proto/iris.proto /seldon-data/rpc/jar/iris.jar
{% endhighlight %}

The second step to inform Seldon of this can be carried out via the seldon-cli with the following command where we pass the jar location and the class name.

{% highlight bash %}
seldon-cli rpc --action set --client-name test --jar /seldon-data/rpc/jar/iris.jar --request-class io.seldon.microservice.iris.IrisPredictRequest
{% endhighlight %}


## Launch gRPC microservice
We can now launch our microservice using the script [start-microservice](scripts.html#start-microservice)
{% highlight bash %}
start-microservice --type prediction --client test -i xgboostrpc seldonio/iris_xgboost_rpc:2.1 rpc 1.0
{% endhighlight %}

This will deploy our microservice and inform Seldon of where it is running so requests can be sent to it.

## Test via REST or gRPC clients
We can test our microservice via either the REST or gRPC endpoints of Seldon. To test the REST endpoint we can use the seldon-cli

{% highlight bash %}
seldon-cli --quiet api --client-name test --endpoint /js/predict --json '{"data":{"f1":1,"f2":2.7,"f3":5.3,"f4":1.9}}'
{% endhighlight %}

This should produce a result like:

{% highlight json %}
{
  "meta": {
    "puid": "f91b158ba046d438cfea82aff4c382f996f5bf51",
    "modelName": "model_xgb",
    "variation": "xgboostrpc"
  },
  "predictions": [
    {
      "prediction": 0.0025230366736650467,
      "predictedClass": "Iris-setosa",
      "confidence": 0.0025230366736650467
    },
    {
      "prediction": 0.0035000948701053858,
      "predictedClass": "Iris-versicolor",
      "confidence": 0.0035000948701053858
    },
    {
      "prediction": 0.9939768314361572,
      "predictedClass": "Iris-virginica",
      "confidence": 0.9939768314361572
    }
  ],
  "custom": null
}
{% endhighlight %}

To test the external gRPC endpoint we will need to create a gRPC client. To authenticate we will need to get an Oauth token using the Seldon REST endpoint.

There is an example client for the Iris prediction in [docker/examples/iris/xgboost_rpc/python/iris_rpc_client.py](https://github.com/SeldonIO/seldon-server/tree/master/docker/examples/iris/xgboost_rpc/python/iris_rpc_client.py) shown below. You will need to have installed the pyseldon package to get the seldon rpc compiled code.

{% highlight python %}
import os
import sys, getopt, argparse
import logging
import json
import grpc
import iris_pb2
import seldon.rpc.seldon_pb2 as seldon_pb2
from google.protobuf import any_pb2
import requests

class IrisRpcClient(object):

    def __init__(self,host,http_transport,http_port,rpc_port):
        self.host = host
        self.http_transport = http_transport
        self.http_port = http_port
        self.rpc_port = rpc_port


    def getToken(self,key,secret):
        params = {}
        params["consumer_key"] = key
        params["consumer_secret"] = secret
        url = self.http_transport+"://"+self.host+":"+str(self.http_port)+"/token"
        r = requests.get(url,params=params)
        if r.status_code == requests.codes.ok:
            j = json.loads(r.text)
            return j["access_token"]
        else:
            print "failed call to get token"
            return None

    def callRpc(self,token):
        channel = grpc.insecure_channel(self.host+':'+str(self.rpc_port))
        stub = seldon_pb2.ClassifierStub(channel)

        data = iris_pb2.IrisPredictRequest(f1=1.0,f2=0.2,f3=2.1,f4=1.2)
        dataAny = any_pb2.Any()
        dataAny.Pack(data)
        meta = seldon_pb2.ClassificationRequestMeta(puid="12345")
        request = seldon_pb2.ClassificationRequest(meta=meta,data=dataAny)
        metadata = [(b'oauth_token', token)]
        reply = stub.Predict(request,999,metadata=metadata)
        print reply

if __name__ == '__main__':
    import logging
    logger = logging.getLogger()
    logging.basicConfig(format='%(asctime)s : %(levelname)s : %(name)s : %(message)s', level=logging.DEBUG)
    logger.setLevel(logging.INFO)

    parser = argparse.ArgumentParser(prog='create_replay')
    parser.add_argument('--host', help='rpc server host', default="localhost")
    parser.add_argument('--http-transport', help='http or https', default="http")
    parser.add_argument('--http-port', help='http server port', type=int, default=30015)
    parser.add_argument('--rpc-port', help='rpc server port', type=int, default=30017)
    parser.add_argument('--key', help='oauth consumer key')
    parser.add_argument('--secret', help='oauth consumer secret')

    args = parser.parse_args()
    opts = vars(args)
    rpc = IrisRpcClient(host=args.host,http_transport=args.http_transport,http_port=args.http_port,rpc_port=args.rpc_port)
    token = rpc.getToken(args.key,args.secret)
    if not token is None:
        print token
        rpc.callRpc(token)
    else:
        print "failed to get token"
{% endhighlight %}

The code will
  1. get an Oauth token
  2. call the grpc endpoint

You will need the seldon host and ports for the http and grpc endpoints and the key and secret for your client.
To get the key and secret for client "test" run:

{% highlight bash %}
seldon-cli keys --action list --client-name test
{% endhighlight %}

An example call for this client might be:

{% highlight bash %}
python iris_rpc_client.py --host localhost --http-port 30015 --key B3ZH3AFANDMX65VX6YPS --secret LGD1K4D7TN07H0OLQT4B
{% endhighlight %}

which should produce a similar result to the REST call above.

{% highlight bash %}
meta {
  modelName: "model_xgb"
  variation: "xgboostrpc"
}
predictions {
  prediction: 0.992686629295
  predictedClass: "Iris-setosa"
  confidence: 0.992686629295
}
predictions {
  prediction: 0.00392687413841
  predictedClass: "Iris-versicolor"
  confidence: 0.00392687413841
}
predictions {
  prediction: 0.00338655733503
  predictedClass: "Iris-virginica"
  confidence: 0.00338655733503
}
custom {
}
{% endhighlight %}





 