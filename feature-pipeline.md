---
layout: default
title: Feature Pipelines
---

##### General Prediction Steps 

 [setup server](/seldon-server-setup.html) --> [events](prediction-api.html) --> **feature extraction pipeline** --> [offline model](offline-prediction-models.html) --> [runtime scorer](/runtime-prediction.html) --> [microservice scorer](/pluggable-prediction-algorithms.html) --> [predictions](prediction-api.html)


# Predictive Pipelines 
Feature extraction pipelines allow you to define a repeatable process to transform a set of input features before you build a machine learning model on a final set of features. When the resulting model is put into production the feature pipeline will need to be rerun on each input feature set before being passed to the model for scoring.

Seldon feature pipelines are presently available in python. We plan to provide Spark based pipelines in the future.

## Python modules
Seldon provides a set of [python modules](python-package.html) to help construct feature pipelines for use inside Seldon. We use [scikit-learn](http://scikit-learn.org/stable/) pipelines and [Pandas](http://pandas.pydata.org/). For feature extraction and transformation we provide a starter set of python scikit-learn Tranformers that take Pandas dataframes as input apply some transformations and output Pandas dataframes. 

The currently available example transforms are:

 * **Include_features_transform** : include a subset of features
 * **Exclude_features_transform** : exclude some subset of features
 * **Binary_transform** : create a binary feature based on existence of a feature
 * **Split_transform** : split a series of textual features into tokens
 * **Exist_features_transform** : filter data to only those containing a set of features
 * **Svmlight_transform** : create a feature that contains SVMLight numeric features from some input set of features
 * **Feature_id_transform** : create an id feature from some input feature
 * **Tfidf_transform** : create TFIDF features from an input feature
 * **Auto_transform** : attempt to automatically normalize and create numeric, categorical and date features
 * **sklearn_transform** : apply a [sklearn Transformer](http://scikit-learn.org/stable/data_transforms.html) to a Pandas Dataframe

## Creating a Machine Learning model
As a final stage of any pipeline you would usually add a scikit learn Estimtor. We provide 3 builtin Estimators which allow Pandas dataframes as input and a general Estimator that can take any sckit-learn compatible estimator.

 * **XGBoostClassifier** : XGBoost classifier which allows Pandas Dataframes as input
 * **VWClassifier** : VW classifier which allows Pandas Dataframes as input
 * **KerasClassifier** : Keras classifier which allows Pandas Dataframes as input
 * **SKLearnClassifier** : General classifier that runs any [sklearn classifier](http://scikit-learn.org/stable/supervised_learning.html) taking Pandas dataframes as input.

# Simple Predictive Pipeline using Iris Dataset
An example pipeline to do very simple extraction on the Iris dataset is contained within the code at `external/predictor/python/docker/examples/iris`. This contains pipelines that utilize Seldon's Docker pipeline and create the following python pipelines:

 1. Create an id feature from the name feature
 1. Create an SVMLight feature from the four core predictive features
 1. Create a model with either XGBoost, Vowpal Wabbit or Keras

The pipeline utilizing XGBoost is shown below

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

The example is explained in more detail [here](iris-demo.html)




