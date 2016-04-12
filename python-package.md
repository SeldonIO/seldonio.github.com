---
layout: default
title: python module
---

# Python modules
Seldon provides a set of python modules to help construct feature pipelines for use inside Seldon.

 * [Package overview](prediction-pipeline.html)
 * [Seldon package reference](/python/index.html)

You can install the python modules in three ways:

 * [Docker](#docker)
 * [pip](#pip)
 * [python setup.py](#python-setup)


## Docker<a name="docker"></a>

A Docker image contains the python modules and all dependencies needed. 

{% highlight bash %}
   docker pull seldonio/pyseldon:1.11
{% endhighlight %}

## Local build software

 * There are dependencies for local software for the python/pip install to run successfully as well as dependencies for the libraries such as vowpal wabbit. For debian based systems these would be satisfied with:
{% highlight bash %}
apt-get update
apt-get install build-essential automake autoconf libxmu-dev g++ gcc libpthread-stubs0-dev libtool libboost-program-options-dev libboost-python-dev zlib1g-dev libc6 libgcc1 libstdc++6 libblas-dev liblapack-dev git telnet procps memcached libmemcached-dev
{% endhighlight %}


## Pip<a name="pip"></a>

Two custom libraries are needed - a Seldon fork of wabbit_wappa and BayesianOptimization if you wish to optimize hyper parameters in pipeline estimators.

 1. check you have the local software needed as given above
 1. ```pip install -e git+git://github.com/SeldonIO/wabbit_wappa#egg=wabbit-wappa-3.0.2```
 1. ```pip install -e git+git://github.com/fmfn/BayesianOptimization#egg=bayes_opt```
 1. ```pip install seldon```


## Python install<a name="python-setup"></a>

 * check you have the local software needed as given above
 * If you want to use the module bayes_opt then 
    {% highlight bash %}
      pip install -e git+git://github.com/fmfn/BayesianOptimization#egg=bayes_opt
    {% endhighlight %}
 * Go to  ```python``` folder of the seldon project and run
{% highlight bash %}
 python setup.py install
{% endhighlight %}

### Module Dependencies

The core module dependencies are:

  * [pandas](http://pandas.pydata.org/) **version >= 0.17**
  * [scikit-learn](http://scikit-learn.org/stable/) **version >= 0.17**
  * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki)
  * [a Seldon fork of wabbit_wappa](https://github.com/SeldonIO/wabbit_wappa)
  * [xgboost](https://github.com/dmlc/xgboost)
  * [unicodecsv](https://github.com/jdunck/python-unicodecsv)
  * [kazoo](https://kazoo.readthedocs.org/en/latest/)
  * [boto](https://github.com/boto/boto)
  * [keras](https://github.com/fchollet/keras)
  * [gensim](https://radimrehurek.com/gensim/)
  * [annoy](https://github.com/spotify/annoy)
  * [Flask](http://flask.pocoo.org/)

The above packages themselves have many dependencies so if you are starting from scratch it may be best to install [Anaconda](http://continuum.io/downloads) which will provide many of the dependencies. Depending on the version of anaconda you install you may need to upgrade some packages to their latest, e.g.

{% highlight bash %}
pip install pandas --upgrade
pip install sklearn --upgrade
{% endhighlight %}

You should install the local software needed as gven above or follow the steps in the [pyseldon Dockerfile](https://github.com/SeldonIO/seldon-server/blob/master/python/docker/pyseldon/Dockerfile) which gives install commands of how to satisfy the dependencies if you wish to run a local custom install. The Dockerfile is inherited fron anaconda which uses a debian base image. For other linux based systems the commands to get the correct software may vary.




