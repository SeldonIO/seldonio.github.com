---
layout: default
title: python module
---

# Python modules
Seldon provides a set of python modules to help construct feature pipelines for use inside Seldon.

 * [Seldon Package Reference](/python/index.html)

You can install them in three ways:

## Dependencies

The module dependencies are:

  * [scikit_learn](http://scikit-learn.org/stable/)
  * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki)
  * [a Seldon fork of wabbit_wappa](https://github.com/SeldonIO/wabbit_wappa)
  * [xgboost](https://github.com/dmlc/xgboost)
  * [unicodecsv](https://github.com/jdunck/python-unicodecsv)
  * [kazoo](https://kazoo.readthedocs.org/en/latest/)
  * [boto](https://github.com/boto/boto)
  * [keras](https://github.com/fchollet/keras)

The above packages themselves have many dependencies so if you are starting from scratch it may be best to install [Anaconda](http://continuum.io/downloads) which will provide many of the dependencies.

## Python install

 * If you want to use the module bayes_opt then ``````pip install -e git+git://github.com/fmfn/BayesianOptimization#egg=bayes_opt```

Go to  ```external/predictor/python``` and run
{% highlight bash %}
 python setup.py install
{% endhighlight %}

## Docker

A Docker image contains the pipelines and dependencies needed. It can be used as a base to build your pipelines from within Docker.

{% highlight bash %}
   docker pull seldonio/pipelines:1.2
{% endhighlight %}

## Pip

Two custom libraries are needed - a Seldon fork of wabbit_wappa and BayesianOptimization if you wish to optimize hyper parameters in pipeline estimators.

 1. ```pip install -e git+git://github.com/SeldonIO/wabbit_wappa#egg=wabbit-wappa-3.0.2```
 1. ```pip install -e git+git://github.com/fmfn/BayesianOptimization#egg=bayes_opt```
 1. ```pip install seldon```



