---
layout: default
title: Feature Pipelines
---

# Feature Extraction Pipelines 
Feature pipelines allow you to define a repeatable process to transform a set of input features before you build a machine learning model on a final set of features. When the resulting model is put into production the feature peipeline will need to be rerun on each input feature set before being passed to the model for scoring.

Seldon feature pipelines are presently available in python. We plan to provide Spark based pipelines in the future.

# Python modules
Seldon provides a set of [python modules](python-package.html) to help construct feature pipelines for use inside Seldon.

A pipeline consists of a series of Feature_Transforms. The currently available transforms are:

 * **Include_features_transform** : include a subset of features
 * **Split_transform** : split a series of textual features into tokens
 * **Exist_features_transform : filter data to only those containing a set of features
 * **Svmlight_transform** : create a feature that contain SVMLight numeric features from some input set of features
 * **Feature_id_transform** : create an id feature some input feature
 * **Tfidf_transform** : create TFIDF features from an input feature
 * **Auto_transform** : attempt to automatically normalize and create numeric, categorical and date features

# Simple Example
An example pipeline to do very simple extraction on the Iris dataset is contained within the code at `external/predictor/python/docker/examples/iris`. This contains a pipeline that extends Seldon's Docker pipeline base with the following python pipeline:

 1. Create an id feature from the name feature
 1. Create an SVMLight feature from the four core predictive features

{% highlight python %}
import sys, getopt, argparse
import seldon.pipeline.basic_transforms as bt
import seldon.pipeline.pipelines as pl
import sys

def run_pipeline(events,features,models):
    p = pl.Pipeline(input_folders=events,output_folder=features,local_models_folder="models_tmp",models_folder=models)

    tNameId = bt.Feature_id_transform(min_size=0,exclude_missing=True)
    tNameId.set_input_feature("name")
    tNameId.set_output_feature("nameId")
    svmTransform = bt.Svmlight_transform(included=["f1","f2","f3","f4"] )
    svmTransform.set_output_feature("svmfeatures")
    p.add(tNameId)
    p.add(svmTransform)
    p.fit_transform()

if __name__ == '__main__':
    parser = argparse.ArgumentParser(prog='bbm_pipeline')
    parser.add_argument('--events', help='events folder', required=True)
    parser.add_argument('--features', help='output features folder', required=True)
    parser.add_argument('--models', help='output models folder', required=True)

    args = parser.parse_args()
    opts = vars(args)

    run_pipeline([args.events],args.features,args.models)
{% endhighlight %}

The example code provides methods to download the Iris dataset and create the JSON events. Then the pipeline can be run as:

{% highlight bash %}
docker run --rm -t -v ${PWD}/data:/data iris_pipeline bash -c "python /pipeline/iris_pipeline.py --events /data/iris/events/1 --features /data/iris/features/1 --models /data/iris/models/1"
{% endhighlight %}

The first few lines of the input events JSON look like:

{% highlight json %}
{"f1": 6.1, "f2": 3.0, "f3": 4.9, "f4": 1.8, "name": "Iris-virginica"}
{"f1": 5.0, "f2": 3.2, "f3": 1.2, "f4": 0.2, "name": "Iris-setosa"}
{"f1": 5.7, "f2": 2.9, "f3": 4.2, "f4": 1.3, "name": "Iris-versicolor"}
{"f1": 5.2, "f2": 2.7, "f3": 3.9, "f4": 1.4, "name": "Iris-versicolor"}
{% endhighlight %}

This is transformed by the pipeline into:

{% highlight json %}
{"f1": 6.1, "f2": 3.0, "f3": 4.9, "f4": 1.8, "name": "Iris-virginica", "nameId": 1, "svmfeatures": {"1": 6.1, "2": 3.0, "3": 4.9, "4": 1.8}}
{"f1": 5.0, "f2": 3.2, "f3": 1.2, "f4": 0.2, "name": "Iris-setosa", "nameId": 2, "svmfeatures": {"1": 5.0, "2": 3.2, "3": 1.2, "4": 0.2}}
{"f1": 5.7, "f2": 2.9, "f3": 4.2, "f4": 1.3, "name": "Iris-versicolor", "nameId": 3, "svmfeatures": {"1": 5.7, "2": 2.9, "3": 4.2, "4": 1.3}}
{"f1": 5.2, "f2": 2.7, "f3": 3.9, "f4": 1.4, "name": "Iris-versicolor", "nameId": 3, "svmfeatures": {"1": 5.2, "2": 2.7, "3": 3.9, "4": 1.4}}
{% endhighlight %}

# Advanced Example
The following reasonably complex example create a pipeline from training data and runs the same series of transformations on test data. The input events files should contain JSON data.

The pipeline does the following transformations:

 1. Keeps only rows containing a likeids feature
 1. Filters features to only a subset using the Include_features_transform
 1. Splits 3 textual features to make a feature containing tokens
 1. Creates an id feature from the "group" feature, keeping only those that appear at least 200 times in the training data
 1. Creates an id feature from the "category" feature
 1. Creates a TFIDF feature, doing a chi-squared test against the groupId feature created above
 1. Create a SVM feature from the the like TFIDF feature

For the testing data the pipeline is reloaded and run against the test data to perform the same transformations.

{% highlight python %}
import sys, getopt, argparse
import seldon.pipeline.basic_transforms as bt
import seldon.pipeline.tfidf_transform as ptfidf
import seldon.pipeline.pipelines as pl

def training_pipeline(events,features,models,awskey,awssecret):
    p = pl.Pipeline(input_folders=events,output_folder=features,local_models_folder="models_train",models_folder=models,aws_key=awskey,aws_secret=awssecret)

    tExist = bt.Exist_features_transform(included=['likeids'])
    tFilter = bt.Include_features_transform(included=["likeids","group","category","utm_source","utm_medium","utm_campaign","friend_uuids"])
    tSplit = bt.Split_transform(split_expression=" |\-|\_",ignore_numbers=True,input_features=["utm_source","utm_medium","utm_campaign"])
    tSplit.set_output_feature("campaign")
    tGroupId = bt.Feature_id_transform(min_size=200,exclude_missing=True)
    tGroupId.set_input_feature("group")
    tGroupId.set_output_feature("groupId")
    tCatId = bt.Feature_id_transform(min_size=0)
    tCatId.set_input_feature("category")
    tCatId.set_output_feature("categoryId")
    tTfidf = ptfidf.Tfidf_transform(select_features=True,target_feature="groupId")
    tTfidf.set_input_feature("likeids")
    tTfidf.set_output_feature("likeid_tfidf")
    svmTransform = bt.Svmlight_transform(included=["likeid_tfidf"] )
    svmTransform.set_output_feature("svmfeatures")
    tFilterFinal = bt.Include_features_transform(included=["likeid_tfidf","svmfeatures","groupId","group"])
    p.add(tExist)
    p.add(tFilter)
    p.add(tSplit)
    p.add(tGroupId)
    p.add(tCatId)
    p.add(tTfidf)
    p.add(svmTransform)
    p.add(tFilterFinal)
    p.fit_transform()

def testing_pipeline(events,features,models,awskey,awssecret):
    p = pl.Pipeline(input_folders=events,output_folder=features,local_models_folder="models_test",models_folder=models,aws_key=awskey,aws_secret=awssecret)
    p.transform()
    p.store_features()


if __name__ == '__main__':
    parser = argparse.ArgumentParser(prog='pipeline_example')
    parser.add_argument('--train_events', help='training events folder', required=True)
    parser.add_argument('--train_features', help='output features folder', required=True)
    parser.add_argument('--test_events', help='testing events folder', required=True)
    parser.add_argument('--test_features', help='output features folder', required=True)
    parser.add_argument('--train_models', help='output models folder', required=True)
    parser.add_argument('--aws_key', help='aws key', default=None)
    parser.add_argument('--aws_secret', help='aws secret', default=None)

    args = parser.parse_args()
    opts = vars(args)

    training_pipeline([args.train_events],args.train_features,args.train_models,args.aws_key,args.aws_secret)
    testing_pipeline([args.test_events],args.test_features,args.train_models,args.aws_key,args.aws_secret)


{% endhighlight %}



