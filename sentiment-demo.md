---
layout: default
title: Sentiment Prediction Demo
---

# Sentiment Prediction Demo

A worked step-by-step example using the [Amazon Fine Foods Review](http://snap.stanford.edu/data/web-FineFoods.html) dataset is provided here. The dataset provides around 500,000 reviews and ratings from Amazon for fine food products. We will use it to create a sentiment predictor that takes the review text and predicts whether it is a positive or negative review. A model will be created and run as a microservice within Seldon.


 * [Prerequisites](#prerequisites)
 * [Create Model](#model)
 * [Serve predictions](#predictions)
 * [Detailed Steps](#detailed-steps)

# Prerequisites<a name="prerequisites"></a>

 * [You have installed Seldon on a Kubernetes cluster](install.html) with Spark included (Spark is included by default).
 * You haved added ```seldon-server/kubernetes/bin``` to you shell PATH environment variable.



# **Create Model**<a name="model"></a>

The model creation step can be accomplished by running the Kubernetes deployment in ```kubernetes/conf/examples/finefoods/train-finefoods.json``` which should have been created when you followed the [install and configuration steps](install.html).

Create the kubernetes job:

{% highlight bash %}
cd kubernetes/conf/examples/finefoods
kubectl create -f train-finefoods.json
{% endhighlight %}

The job may take a 10-20 minutes to fully run. You can check its status with ```kubectl get jobs -l job-name=train-finefoods```. 

This job will:

 * Download the finefoods review data. (N.B. make sure your Kubernetes cluster has access to the internet)
 * Create a dataset where reviews with a score greater than 3 are marked as "positive" and less than 3 as "negative". Review with a score of 3 are ignored.
 * Create an XGBoost classification model using TFIDF features
 * Save the modelling pipeline to persistent storage


# **Serve Predictions**<a name="predictions"></a>

To serve predictions we will load the saved pipeline into a microservice. This can be accomplished by using the script ```start-microservice``` in ```seldon-server/kubernetes/bin```.

{% highlight bash %}
start-microservice --type prediction --client test -p finefoods-xgboost /seldon-data/seldon-models/finefoods/1/ 1.0
{% endhighlight %}

This will load the pipeline saved in /seldon-data/seldon-models/finefoods/1/ and create a single replica microservice called finefoods-xgboost. It will activate this for the "test" client as a prediction algorithm.

You can then test the predictions, e.g.:

{% highlight bash %}
seldon-cli api --client-name test --endpoint /js/predict --json '{"data":{"review":"The candy is just red.   No flavor. Just  plan and chewy.  I would never buy them again."}}'
{% endhighlight %}

Which should produce a result like below indication a "negative" sentiment:

{% highlight json %}
{
  "size": 2,
  "requested": 0,
  "list": [
    {
      "prediction": 0.594055,
      "predictedClass": "0",
      "confidence": 0.594055
    },
    {
      "prediction": 0.405945,
      "predictedClass": "1",
      "confidence": 0.405945
    }
  ]
}
{% endhighlight %}

While the following:

{% highlight bash %}
seldon-cli api --client-name test --endpoint /js/predict --json '{"data":{"review":"My daughter loves twizzlers and this shipment of six pounds really hit the spot"}}'
{% endhighlight %}

Should produce a "positive" sentiment:

{% highlight json %}
{
  "size": 2,
  "requested": 0,
  "list": [
    {
      "prediction": 0.268916,
      "predictedClass": "0",
      "confidence": 0.268916
    },
    {
      "prediction": 0.731084,
      "predictedClass": "1",
      "confidence": 0.731084
    }
  ]
}
{% endhighlight %}

# **Detailed Steps**<a name="detailed-steps"></a>
The model creation steps are provided by the Docker image seldonio/finefoods_xgboost whose code can be found in ```seldon-server/docker/examples/finefoods```. There is a script create_model.sh which is shown below:

{% highlight bash %}
#!/bin/bash

set -o nounset
set -o errexit

if [ "$#" -ne 2 ]; then
    echo "need <model_folder> <sample in range 0.0 to 1.0>"
    exit -1
fi

STARTUP_DIR="$( cd "$( dirname "$0" )" && pwd )"
MODEL_FOLDER=$1
SAMPLE=$2

function get_data {
    
    cd ${STARTUP_DIR}
    echo "nameserver 8.8.8.8" >> /etc/resolv.conf
    wget http://snap.stanford.edu/data/finefoods.txt.gz
    gunzip finefoods.txt.gz
    iconv -f iso-8859-1 -t utf-8 finefoods.txt -o finefoods_utf8.txt
    mkdir -p data
    echo "sentiment,review" > data/train.csv
    cat finefoods_utf8.txt | awk 'BEGIN{OFS=","}/^review\/score/{if ($2<3){s=0} else if ($2>3){s=1}else{s=-1}}/^review\/text/{if (s>-1){gsub(/,/," ");print s,$0}}' | sed -e 's/review\/text://g' -e 's/<br \/>//g' -e 's/\./ /g' >> data/train.csv

}

function train {

    mkdir ${STARTUP_DIR}/folds
    python ${STARTUP_DIR}/create_model.py --data ${STARTUP_DIR}/data --model ${MODEL_FOLDER} --sample ${SAMPLE}
    perf -SEN -SPC -NPV -PPV -ACC -confusion -files folds/1_correct.txt <(cut -d' ' -f2 folds/1_predictions_proba.txt)
}


get_data

train
{% endhighlight %}

The script:

 * Downloads the data, uncompresses and turns it into UTF-8
 * creates a data set from review and sentiment based on score
 * Runs the create_model.py python script to create the model
 * prints the confusion matrix from one of the cross validation fold runs.

The model creation in create_model.py  uses the pyseldon package to build a sklearn pipeline as show below:

{% highlight python %}
        tTfidf = ptfidf.Tfidf_transform(input_feature="review",output_feature="tfidf",target_feature="sentiment",min_df=10,max_df=0.7,select_features=False,topn_features=50000,stop_words="english",ngram_range=[1,2])


        tFilter2 = bt.Include_features_transform(included=["tfidf","sentiment"])

        svmTransform = bt.Svmlight_transform(output_feature="svmfeatures",excluded=["sentiment"],zero_based=False)

        classifier_xg = xg.XGBoostClassifier(target="sentiment",svmlight_feature="svmfeatures",silent=1,max_depth=5,n_estimators=200,objective='binary:logistic',scale_pos_weight=0.2)

        cv = cf.Seldon_KFold(classifier_xg,metric='auc',save_folds_folder="./folds")
    
        transformers = [("tTfidf",tTfidf),("tFilter2",tFilter2),("svmTransform",svmTransform),("cv",cv)]

        p = Pipeline(transformers)
{% endhighlight %}

 * Create TFIDF features using the pyseldon tfidf transformer, selecting the top 50K features from unigram and bigrams of the text.
 * Transform the features into SVMlight format for ease of use by XGBoost
 * Add an XGBoost classifier using th pyseldon wrapper with positive scaled weight of 0.2 to handle the data imbalance where positive reviews out number neagtive reviews 5:1.
 * Run 5 fold cross validation
 * Combine these into a sklearn pipeline

The script saves the pipleline after modelling so that it can be reloaded and run in prediction mode to score new data.




