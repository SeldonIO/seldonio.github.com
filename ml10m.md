---
layout: default
title: Movielens 10 Million Demo
---

# Movielens 10 Million Example

A worked step-by-step example using the [Movielens 10 Million](http://grouplens.org/datasets/movielens/10m/) dataset is provided here. 

 * [Prerequisites](#prerequisites)
 * [Running](#running)
 * [Detailed Steps](#detailed-steps)

# Prerequisites<a name="prerequisites"></a>

 * [You have installed Seldon on a Kubernetes cluster](install.html) with Spark included (Spark is included by default).
 * You haved added ```seldon-server/kubernetes/bin``` to you shell PATH environment variable.

# **Running**<a name="running"></a>

The entire set of steps can be executed by running one of the Kubernetes deployments in ```seldon-server/kubernetes/conf/examples/ml10m``` which should have been created when you followed the [install and configuration steps](install.html).

There are two jobs:

 * ml10m-import-item-similarity.json : Downloads the data, and create an item-similarity model
 * ml10m-import-matrix-factorization.json : Downloads the data, and create an matrix factorization model

Start the chosen the kubernetes job, for example:

{% highlight bash %}
cd seldon-server/kubernetes/conf/examples/ml10m
kubectl create -f ml10m-import-item-similarity.json
{% endhighlight %}

The job may take 10 or more minutes to run depending on the size and compute power of your Kubernetes cluster and the Spark cluster within it. You can check its status with ```kubectl get jobs -l job-name=ml10m-import```. 

This job will:

 * Download the movielens 10 million data. (N.B. make sure your Kubernetes cluster has access to the internet)
 * Create a new client "ml10m", create the item meta-data schema and import user, and items.
 * Create a historical actions dataset from the movie ratings
 * Run a Spark job via Luigi to create a model
 * Setup a runtime scorer

You can then test the recommendations by doing:

{% highlight bash %}
seldon-cli api --client-name ml10m --endpoint /js/recommendations --item 50 --limit 4 --user 625
{% endhighlight %}

The above gets recommendations based on a recent action history for user 625  being movie 50 which is "The Usual Suspects". The result should look something like:

{% highlight json %}
{
  "size": 4,
  "requested": 4,
  "list": [
    {
      "id": "318",
      "name": "",
      "type": 1,
      "first_action": 1465573742000,
      "last_action": 1465573742000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "1",
        "title": "Shawshank Redemption, The (1994)"
      }
    },
    {
      "id": "296",
      "name": "",
      "type": 1,
      "first_action": 1465573742000,
      "last_action": 1465573742000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "1",
        "title": "Pulp Fiction (1994)"
      }
    },
    {
      "id": "593",
      "name": "",
      "type": 1,
      "first_action": 1465573742000,
      "last_action": 1465573742000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "1",
        "title": "Silence of the Lambs, The (1991)"
      }
    },
    {
      "id": "47",
      "name": "",
      "type": 1,
      "first_action": 1465573742000,
      "last_action": 1465573742000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "1",
        "title": "Seven (a.k.a. Se7en) (1995)"
      }
    }
  ]
}

{% endhighlight %}

# **Detailed Steps**<a name="detailed-steps"></a>

Below are the deatiled steps which can be found [here](https://github.com/SeldonIO/seldon-server/blob/master/docker/examples/ml10m/create_ml10m_recommender.sh).

## Download Data

 * Hack to ensure we have a namesever for external DNS (seems to be required for local Docker running of Kubernetes)
 * Download and unzip Movielens data
 * Convert to UTF-8 the item meta-data

{% highlight bash %}
    echo "nameserver 8.8.8.8" >> /etc/resolv.conf
    wget http://files.grouplens.org/datasets/movielens/ml-10m.zip
    unzip ml-10m.zip
    iconv -f iso-8859-1 -t utf-8 ml-10M100K/movies.dat -o ml-10M100K/movies.dat.utf8
{% endhighlight %}

## Create Historical Data Files

 * Create item, user and action CSV files from raw data

{% highlight bash %}
    echo "create items csv"
    cat <(echo 'id,title') <(cat ml-10M100K/movies.dat.utf8 | awk -F '::' '{printf("%d,\"%s\"\n",$1,$2)}') > items.csv

    echo "create users csv"
    cat <(echo "id") <(cat ml-10M100K/ratings.dat | awk -F'::' '{print $1}' | uniq) > users.csv

    echo "create actions csv"
    cat <(echo "user_id,item_id,value,time") <(cat ml-10M100K/ratings.dat | awk -F'::' 'BEGIN{OFS=","}{print $1,$2,$3,$4}') > actions.csv
{% endhighlight %}

## Create Client 

We will use a item schema to hold the title of the movies, as show below

{% highlight json %}
{
    "types": [
        {
            "type_attrs": [
                {
                    "name": "title",
                    "value_type": "string"
                }
		    ],
            "type_id": 1,
            "type_name": "Movies"
	    }
    ]
}
{% endhighlight %}

The steps to setup and import the data can be done via the Seldon CLI

 * Create a new ml10m client
 * Setup the schema using above JSON
 * Import the users and items as defined by CSV files
 * Create actions file from CSV file

{% highlight bash %}

    seldon-cli client --action setup --client-name ml10m --db-name ClientDB
    kubectl cp attr.json /seldon-control:/tmp/attr.json && seldon-cli attr --action apply --client-name ml10m --json /tmp/attr.json
    kubectl cp items.csv /seldon-control:/tmp/items.csv && seldon-cli import --action items --client-name ml10m --file-path /tmp/items.csv
    kubectl cp users.csv /seldon-control:/tmp/users.csv && seldon-cli import --action users --client-name ml10m --file-path /tmp/users.csv
    kubectl cp actions.csv /seldon-control:/tmp/actions.csv && seldon-cli import --action actions --client-name ml10m --file-path /tmp/actions.csv

{% endhighlight %}

## Build a Recommendation Model

The script contains calls to build either a item-similarity model or a matrix factorization model using luigi:

{% highlight bash %}
    rm -rf /seldon-data/seldon-models/ml10m/matrix-factorization/1
    luigi --module seldon.luigi.spark SeldonMatrixFactorization --local-schedule --client ml10m --startDay 1 

    rm -rf /seldon-data/seldon-models/ml10m/item-similarity/1
    luigi --module seldon.luigi.spark SeldonItemSimilarity --local-schedule --client ml10m --startDay 1 --ItemSimilaritySparkJob-sample 0.25 --ItemSimilaritySparkJob-dimsumThreshold 0.5 --ItemSimilaritySparkJob-limit 100 
{% endhighlight %}

## Setup Runtime Scorer

We setup an appropriate runtime scorer depending on which model we created, for the item-similarity we would do:

{% highlight bash %}
    cat <<EOF | seldon-cli rec_alg --action create --client-name ml10m -f -
{
    "defaultStrategy": {
        "algorithms": [
            {
                "config": [
                    {
                        "name": "io.seldon.algorithm.general.numrecentactionstouse",
                        "value": "1"
                    }
                ],
                "filters": [],
                "includers": [],
                "name": "itemSimilarityRecommender"
            }
        ],
        "combiner": "firstSuccessfulCombiner",
        "diversityLevel": 3
    },
    "recTagToStrategy": {}
}
EOF
    seldon-cli rec_alg --action commit --client-name ml10m
    #pull updated conf from zookeeper so its safe
    seldon-cli client --action zk_pull
{% endhighlight %}
