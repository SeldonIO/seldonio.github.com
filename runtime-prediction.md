---
layout: default
title: Runtime Prediction Algorithms 
---

# Runtime Prediction Algorithms

At runtime Seldon can score requests against prediction models. At present simple classification based Vowpal Wabbit models are supported inside the Seldon Server while the full range of Vowpal Wabbit models can be utilized via a micro-service API. The available runtime prediction algorithms are:

**Runtime Prediction Algorithm** | **Model**
--|--
`vwClassifier` | Vowpal Wabbit binary or one-against-all model
`externalPredictionServer` | [Custom Model](pluggable-prediction-algorithms.html)


These algorithms will be utilized when the prediction endpoints of the [REST](api-oauth.html#predictive-scoring) or [Javascript](api-javascript.html#predictive-scoring) API are used.

## Vowpal Wabbit Classification <a name="vw"></a>

A runtime scorer for Vowpal Wabbit classification models. At present only a subset of VW models are handled, specifically the model needs to be:

 * Saved as a readable model
 * Can be only a binary classification model or "one-against-all", i.e., the only value of the "options:" line in the readable model can be an "--oaa" option.
 * Does not use quadratic, cubic, ngram or other feature extensions of VW

If these requirements are too restrictive one can use an external prediction server accessed over the microservices REST API and utilize a VW server running as a daemon as described in [pluggable prediction algorithms](pluggable-prediction-algorithms.html#prediction-python-vw).

 **Algorithm** : `vwClassifier`  
 **Description** : Utilizes a Vowpal Wabbit classification model   

Example config to set in `/all_clients/[client]/predict_algs`:

{% highlight json %}
 {
  "algorithms":[
   {
   "name":"vwClassifier",
   "config":[]
   }
  ],
 }
{% endhighlight %}


The location of the readable model should be specified at `/all_clients/[client]/vw`. The vwClassifier will attempt to load a file `model` from this location. It will also attempt to load an optional file `classes.txt` which gives the mapping of target class ids to a human friendly name, for example:

{% highlight bash %}
1,Iris-setosa
2,Iris-versicolor
3,Iris-virginica
{% endhighlight %}


