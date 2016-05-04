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
 * [Detailed Steps](#detailed-steps)
 * [Next Steps](#next-steps)

# **Prerequisites**<a name="prerequisites"></a>

 * [You have installed Seldon on a Kubernetes cluster](install.html)
 * You haved added ```seldon-server/kubernetes/bin``` to you shell PATH environment variable.

# **Create a Predictive Model**<a name="model"></a>
The first step is to create a predictive model based on some training data. In this case we have prepacked into 3 Docker images the process of creating a model for the Iris dataset using three popular machine learning toolkits [XGBoost](https://github.com/dmlc/xgboost), [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki) and [Keras](http://keras.io/). Wrappers to call these libraries have been added to our python library to make integrating them as a microservice easy. However, you can build your model using any machine learning library. 

The three images are:

 * seldonio/iris_xgboost : an XGBoost model for the iris dataset
 * seldonio/iris_vw : a VW model for the iris dataset
 * seldonio/iris_keras : a keras model for the iris dataset

For details on building models using our python library see [here](prediction-pipeline.html).

# **Start a microservice**<a name="microservice"></a>

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

The script will use the seldon-cli to update the "test" client to add the microservice as a runtime algorithm. Check with ```kubectl get pods -l name=iris-xgboost-example``` that the pod running the mircroservice is running.  

# **Serve Predictions**<a name="predictions"></a>

You can now call the seldon server using the seldon CLI to test:

{% highlight bash %}
seldon-cli --quiet api --client-name test --endpoint /js/predict --json '{"f1":1,"f2":2.7,"f3":5.3,"f4":1.9}'
{% endhighlight %}

The respone should be like:
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


# **Detailed Steps**<a name="detailed-steps"></a>

The models for the Iris dataset are created using our [pyseldon](python-package.html) library to wrap calls to XGBoost and use Pandas along with a feature transform to normalise the features. The XGBoost version is shown below:

{% highlight python %}
import sys, getopt, argparse
import seldon.pipeline.basic_transforms as bt
import seldon.pipeline.util as sutl
import seldon.pipeline.auto_transforms as pauto
from sklearn.pipeline import Pipeline
import seldon.xgb as xg
import sys

def run_pipeline(events,models):

    tNameId = bt.Feature_id_transform(min_size=0,exclude_missing=True,zero_based=True,input_feature="name",output_feature="nameId")
    tAuto = pauto.Auto_transform(max_values_numeric_categorical=2,exclude=["nameId","name"])
    xgb = xg.XGBoostClassifier(target="nameId",target_readable="name",excluded=["name"],learning_rate=0.1,silent=0)

    transformers = [("tName",tNameId),("tAuto",tAuto),("xgb",xgb)]
    p = Pipeline(transformers)

    pw = sutl.Pipeline_wrapper()
    df = pw.create_dataframe(events)
    df2 = p.fit(df)
    pw.save_pipeline(p,models)


if __name__ == '__main__':
    parser = argparse.ArgumentParser(prog='xgb_pipeline')
    parser.add_argument('--events', help='events folder', required=True)
    parser.add_argument('--models', help='output models folder', required=True)

    args = parser.parse_args()
    opts = vars(args)

    run_pipeline([args.events],args.models)
{% endhighlight %}

To run the runtime scorer for this model we load the model and using the pyseldon wrapper to start a simple Http server.

{% highlight python %}
import sys, getopt, argparse
from seldon.microservice import Microservices

if __name__ == "__main__":
    parser = argparse.ArgumentParser(prog='microservice')
    parser.add_argument('--model_name', help='name of model', required=True)
    parser.add_argument('--pipeline', help='location of prediction pipeline', required=True)
    parser.add_argument('--aws_key', help='aws key', required=False)
    parser.add_argument('--aws_secret', help='aws secret', required=False)

    args = parser.parse_args()
    opts = vars(args)

    m = Microservices(aws_key=args.aws_key,aws_secret=args.aws_secret)
    app = m.create_prediction_microservice(args.pipeline,args.model_name)
    app.run(host="0.0.0.0", debug=True)
{% endhighlight %}

The full source code to create a docker image for model and runtime scorer for the three variants can be found in:

 * ``docker/examples/iris/xgboost``` : an XGBoost model for the iris dataset
 * ```docker/examples/iris/vw``` : a VW model for the iris dataset
 * ```docker/examples/iris/keras``` : a keras model for the iris dataset



# Next Steps<a name="next-steps"></a>

 * [Detailed Guide for General prediction](prediction-guide.html)

