---
layout: default
title: Prediction Example
---

# Iris Classification

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
The first step is to create a predictive model based on some training data. In this case we have prepacked into 4 Docker images the process of creating a model for the Iris dataset using popular machine learning toolkits [XGBoost](https://github.com/dmlc/xgboost), [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki), [Keras](http://keras.io/) and [Scikit-learn](http://scikit-learn.org/stable/). Wrappers to call these libraries have been added to our python library to make integrating them as a microservice easy. However, you can build your model using any machine learning library. 

The four example images are:

 * seldonio/iris_xgboost : an XGBoost model for the iris dataset
 * seldonio/iris_vw : a VW model for the iris dataset
 * seldonio/iris_keras : a keras model for the iris dataset
 * seldonio/iris_scikit : a scikit-learn model for the iris dataset

For details on building models using our python library see [here](prediction-pipeline.html).

# **Start a microservice**<a name="microservice"></a>

At runtime Seldon requires you expose your model scoring engine as a microservice API. In this example case the same image to create the models also exposes it for runtime scoring when run. We can start our chosen microservice using the command line script [start-microservice](scripts.html#start-microservice)

The script creates a Kubernetes deployment for the microservice in ```kubernetes/conf/microservices```. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.

For example to start the XGBoost Iris microservice on the client  "test" (created by seldon-up on startup):

{% highlight bash %}
start-microservice --type prediction --client test -i iris-xgboost seldonio/iris_xgboost:2.1 rest 1.0
{% endhighlight %}

Check with ```kubectl get pods -l name=iris-xgboost``` that the pod running the mircroservice is running.  

# **Serve Predictions**<a name="predictions"></a>

You can now call the seldon server using the seldon CLI to test:

{% highlight bash %}
seldon-cli --quiet api --client-name test --endpoint /js/predict --json '{"data":{"f1":1,"f2":2.7,"f3":5.3,"f4":1.9}}'
{% endhighlight %}

The respone should be like:
{% highlight json %}
{
  "meta": {
    "puid": "242fef28a255cd67d9c56ce6c15f6f0ff65d68e8",
    "modelName": "model_xgb",
    "variation": "iris-xgboost"
  },
  "predictions": [
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
  ],
  "custom": null
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
    import logging
    logger = logging.getLogger()
    logging.basicConfig(format='%(asctime)s : %(levelname)s : %(name)s : %(message)s', level=logging.DEBUG)
    logger.setLevel(logging.INFO)


    parser = argparse.ArgumentParser(prog='microservice')
    parser.add_argument('--model_name', help='name of model', required=True)
    parser.add_argument('--pipeline', help='location of prediction pipeline', required=True)
    parser.add_argument('--aws_key', help='aws key', required=False)
    parser.add_argument('--aws_secret', help='aws secret', required=False)

    args = parser.parse_args()
    opts = vars(args)

    m = Microservices(aws_key=args.aws_key,aws_secret=args.aws_secret)
    app = m.create_prediction_microservice(args.pipeline,args.model_name)

    app.run(host="0.0.0.0", debug=False)
{% endhighlight %}

The full source code to create a docker image for model and runtime scorer for the three variants can be found in:

 * [docker/examples/iris/xgboost](https://github.com/SeldonIO/seldon-server/tree/master/docker/examples/iris/xgboost) : an XGBoost model for the iris dataset
 * [docker/examples/iris/vw](https://github.com/SeldonIO/seldon-server/tree/master/docker/examples/iris/vw) : a VW model for the iris dataset
 * [docker/examples/iris/keras](https://github.com/SeldonIO/seldon-server/tree/master/docker/examples/iris/keras) : a keras model for the iris dataset
 * [docker/examples/iris/scikit](https://github.com/SeldonIO/seldon-server/tree/master/docker/examples/iris/scikit) : a scikit-learn model for the iris dataset



# Next Steps<a name="next-steps"></a>

 * [Detailed Guide for General prediction](prediction-guide.html)

