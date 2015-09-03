---
layout: default
title: Offline Prediction Model Creation
---

# Offline Prediction Model Creation

Seldon currently provide fully integrated classification models built using:

 * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki). 
     * Simple Vowpal Wabbit classification models can be runtime scored inside the [seldon server](runtime-prediction.html) while more complex models can be accessed via our [micro-service API](pluggable-prediction-algorithms.html#prediction-python-vw).
 * [XGBoost](https://github.com/dmlc/xgboost)

These example wrappers can be used as examples to connect other machine learning algorithms to Seldon.

## Create a Vowpal Wabbit Classifier
Create a Vowpal Wabbit classification model from features. We provide a Docker container to run the modelling:

`docker pull seldonio/vw_train`

This image contains a python script `/vw/vw_train/vw_train.py` which provides a light wrapper to the vw module in the Seldon [python modules](python-modules.html). Its takes the following arguments:

 * **--client** : client
 * **--zkHosts** : zookeeper
 * **--inputPath** : input base folder to find features data
 * **--outputPath** : output folder to store model
 * **--day** : days to get features data for
 * **--activate** : activate model in zookeeper
 * **--awsKey** : aws key - needed if input or output is on AWS and no IAM
 * **--awsSecret** : aws secret - needed if input or output on AWS  and no IAM
 * **--vwArgs** : vw training args
 * **--namespaces** :  JSON providing per feature namespace mapping - default is no namespaces
 * **--include** : include these features
 * **--exclude** : exclude these features
 * **--target** : target feature (should contain integer ids in range 1..Num Classes
 * **--target_readable** : the feature containing the human readable version of target
 * **--train_filename** : convert data to vw training format and save to file rather than directly train using wabbit_wappa
 * **--dataType** : json or csv (default json)

The final input and output folders utilize the client name and day to construct the final path. See the [prediction overview](prediction-overview.html) for a summary.

A fully worked example using a pipeline for the Iris data set is contained within the code at `external/predictor/python/docker/examples/iris`. This contains an example to use vowpal wabbit to build a model with the following command using docker:

{% highlight bash %}
docker run --rm -t -v ${PWD}/data:/data seldonio/vw_train bash -c "cd /vw/vw_train ; python vw_train.py --client iris --day 1 --inputPath /data/ --outputPath /data/ --vwArgs '--passes 3 --oaa 3' --target nameId --include f1 f2 f3 f4 --train_filename train.vw --target_readable name"
{% endhighlight %}


## Create an XGBoost Classifier
Create an XGBoost classification model from features. We provide a Docker container to run the modelling:

`docker pull seldonio/xgboost_train`

This image contains a python script `/vw/xgboost_train/xgboost_train.py` which provides a light wrapper to the xgboost module in the Seldon [python modules](python-modules.html). Its takes the following arguments:

 * **--client** : client
 * **--zkHosts** : zookeeper
 * **--inputPath** : input base folder to find features data
 * **--outputPath** : output folder to store model
 * **--day** : days to get features data for
 * **--activate** : activate model in zookeeper
 * **--awsKey** : aws key - needed if input or output is on AWS and no IAM
 * **--awsSecret** : aws secret - needed if input or output on AWS  and no IAM
 * **--svmFeatures** : the JSON feature containing the svm features
 * **--target** : target feature (should contain integer ids in range 1..Num Classes
 * **--target_readable** : the feature containing the human readable version of target

The final input and output folders utilize the client name and day to construct the final path. See the [prediction overview](prediction-overview.html) for a summary.

A fully worked example using a pipeline for the Iris data set is contained within the code at `external/predictor/python/docker/examples/iris`. This contains an example to use XGBoost to build a model with the following command using docker:

{% highlight bash %}
docker run --rm -t -v ${PWD}/data:/data seldonio/xgboost_train bash -c  "cd /xgboost/xgboost_train ; python xgboost_train.py --client iris --inputPath /data --outputPath /data --day 1 --target nameId --svmFeatures svmfeatures --target_readable name"
{% endhighlight %}

