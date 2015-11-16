---
layout: default
title: Feature Pipelines
---

##### General Prediction Steps 

 [setup server](/seldon-server-setup.html) --> [events](prediction-api.html) --> **feature extraction pipeline** --> [runtime scorer](/runtime-prediction.html) --> [microservice scorer](/pluggable-prediction-algorithms.html) --> [predictions](prediction-api.html)


# Predictive Pipelines 
Feature extraction pipelines allow you to define a repeatable process to transform a set of input features before you build a machine learning model on a final set of features. When the resulting model is put into production the feature pipeline will need to be rerun on each input feature set before being passed to the model for scoring.

Seldon feature pipelines are presently available in python. We plan to provide Spark based pipelines in the future.

## Python modules
Seldon provides a set of [python modules](python-package.html) to help construct feature pipelines for use inside Seldon. We use [scikit-learn](http://scikit-learn.org/stable/) pipelines and [Pandas](http://pandas.pydata.org/). For feature extraction and transformation we provide a starter set of python scikit-learn Tranformers that take Pandas dataframes as input apply some transformations and output Pandas dataframes. 

The currently available example transforms are:

 * [Include_features_transform](python/modules/seldon/pipeline/basic_transforms.html#Include_features_transform) : include a subset of features
 * [Exclude_features_transform](python/modules/seldon/pipeline/basic_transforms.html#Exclude_features_transform) : exclude some subset of features
 * [Binary_transform](python/modules/seldon/pipeline/basic_transforms.html#Binary_transform) : create a binary feature based on existence of a feature
 * [Split_transform](python/modules/seldon/pipeline/basic_transforms.html#Split_transform) : split a series of textual features into tokens
 * [Exist_features_transform](python/modules/seldon/pipeline/basic_transforms.html#Exist_features_transform) : filter data to only those containing a set of features
 * [Svmlight_transform](python/modules/seldon/pipeline/basic_transforms.html#Svmlight_transform) : create a feature that contains SVMLight numeric features from some input set of features
 * [Feature_id_transform](python/modules/seldon/pipeline/basic_transforms.html#Feature_id_transform) : create an id feature from some input feature
 * [Tfidf_transform](python/seldon.pipeline.html#module-seldon.pipeline.tfidf_transform) : create TFIDF features from an input feature
 * [Auto_transform](python/seldon.pipeline.html#module-seldon.pipeline.auto_transforms) : attempt to automatically normalize and create numeric, categorical and date features
 * [sklearn_transform](python/seldon.pipeline.html#module-seldon.pipeline.sklearn_transform) : apply a [sklearn Transformer](http://scikit-learn.org/stable/data_transforms.html) to a Pandas Dataframe

### Small Examples

Several small examples can be found in `external/predictor/python/examples`

### Use sklean's StandardScaler on a Pandas DataFrame.

{% highlight python %}
import seldon.pipeline.sklearn_transform as ssk
import pandas as pd
from sklearn.preprocessing import StandardScaler

df = pd.DataFrame.from_dict([{"a":1.0,"b":2.0},{"a":2.0,"b":3.0}])
t = ssk.sklearn_transform(input_features=["a"],output_features=["a_scaled"],transformer=StandardScaler())
t.fit(df)
df_2 = t.transform(df)
print df_2
{% endhighlight %}

When run this would print:

{% highlight bash %}
python sklearn_scaler.py 
   a  b  a_scaled
0  1  2        -1
1  2  3         1
{% endhighlight %}

### Auto transform a set of features

{% highlight python %}
import seldon.pipeline.auto_transforms as auto
import pandas as pd

df = pd.DataFrame([{"a":10,"b":1,"c":"cat"},{"a":5,"b":2,"c":"dog","d":"Nov 13 08:36:29 2015"},{"a":10,"b":3,"d":"Oct 13 10:50:12 2015"}])
t = auto.Auto_transform(max_values_numeric_categorical=2,date_cols=["d"])
t.fit(df)
df2 = t.transform(df)
print df2
{% endhighlight %}

 This would:

 * Turn column "a" into a categorical column due to the small number of variables and limit specified by "max_values_numeric_categorical"
 * Standard scale column "b"
 * Leave column "c" as categorical but convert empty columns to "UKN" category
 * Convert the specified date column "d" to a series of expanded features

 The converted DataFrame would be:

{% highlight bash %}

      a         b    c                   d      d_h1      d_h2 d_hour  \
0  a_10 -1.224745  cat                 NaT       NaN       NaN   hnan   
1   a_5  0.000000  dog 2015-11-13 08:36:29  0.866025 -0.500000     h8   
2  a_10  1.224745  UKN 2015-10-13 10:50:12  0.500000 -0.866025    h10   

       d_m1      d_m2 d_month   d_w      d_w1      d_w2 d_year  
0       NaN       NaN    mnan  wnan       NaN       NaN   ynan  
1 -0.500000  0.866025     m11    w4 -0.433884 -0.900969  y2015  
2 -0.866025  0.500000     m10    w1  0.781831  0.623490  y2015

{% endhighlight %}

## Creating a Machine Learning model
As a final stage of any pipeline you would usually add a scikit learn Estimtor. We provide 3 builtin Estimators which allow Pandas dataframes as input and a general Estimator that can take any sckit-learn compatible estimator.

 * [XGBoostClassifier](python/seldon.html#module-seldon.xgb) : XGBoost classifier which allows Pandas Dataframes as input
 * [VWClassifier](python/seldon.html#module-seldon.vw) : VW classifier which allows Pandas Dataframes as input
 * [KerasClassifier](python/seldon.html#module-seldon.keras) : Keras classifier which allows Pandas Dataframes as input
 * [SKLearnClassifier](python/seldon.html#module-seldon.sklearn_estimator) : General classifier that runs any [sklearn classifier](http://scikit-learn.org/stable/supervised_learning.html) taking Pandas dataframes as input.

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


## Testing and Optimization

There are two modules for helping in testing and optimizing pipelines:

 * [cross_validation](python/seldon.pipeline.html#module-seldon.pipeline.cross_validation) : Allow cross validation to be run on pipelines that use pandas dataframes
 * [bayes_optimize](python/seldon.pipeline.html#module-seldon.pipeline.bayes_optimize) : Optimize an estimators parameters

A notebook showing how to use these can be found in ```external/predictor/python/examples/credit_card.ipynb````


# Further Examples

Further examples can be found in ```external/predictor/python/examples```

 * [wine.ipynb](https://github.com/SeldonIO/seldon-server/blob/master/external/predictor/python/examples/wine.ipynb) : Jupyter python notebook to run a pipeline on wine classification
 * [credit_card.ipynb](https://github.com/SeldonIO/seldon-server/blob/master/external/predictor/python/examples/credit_card.ipynb) : Jupyter python notebook to run a pipeline and optimize on credit card data
 * [sklearn_scaler.py](https://github.com/SeldonIO/seldon-server/blob/master/external/predictor/python/examples/sklearn_scaler.py) : an example of using a sklearn scaler in a pipeline with Pandas
 * [auto_transform.py](https://github.com/SeldonIO/seldon-server/blob/master/external/predictor/python/examples/auto_transform.py) : an example of a simple auto_transform on pandas data




