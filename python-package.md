---
layout: default
title: python module
---

# Python modules
Seldon provides a set of python modules to help construct feature pipelines for use inside Seldon.

 * [Seldon Package Reference](/python/index.html)

You can install them in three ways:

## Install from seldon-server project

The module dependencies are:

  * [scikit_learn](http://scikit-learn.org/stable/)
  * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki)
  * [a Seldon fork of wabbit_wappa](https://github.com/SeldonIO/wabbit_wappa)
  * [xgboost](https://github.com/dmlc/xgboost)
  * [unicodecsv](https://github.com/jdunck/python-unicodecsv)
  * [kazoo](https://kazoo.readthedocs.org/en/latest/)
  * [boto](https://github.com/boto/boto)
  * [keras](https://github.com/fchollet/keras)

The above packages themselves have many dependencies so if you are starting from scratch it may be best to install [Anaconda](http://continuum.io/downloads) which will provide many of the dependencies or use the Docker container below.

After installing the dependencies from a [seldon-server install](seldon-server-setup.html) go to  ```external/predictor/python``` and run
{% highlight bash %}
 python setup.py install
{% endhighlight %}

## Utilize from Docker
A Docker image contains the pipelines and dependencies needed. It can be used as a base to build your pipelines from within Docker.

{% highlight bash %}
   docker pull seldonio/pipelines
{% endhighlight %}


## Install using pip

{% highlight bash %}
   pip install seldon
{% endhighlight %}

**at present you will need to install [xgboost](https://github.com/dmlc/xgboost/tree/master/python-package) and [wabbit_wappa](https://github.com/mokelly/wabbit_wappa) manually as they have issues with their pip packages.**

