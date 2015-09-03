---
layout: default
title: python module
---

# Python modules
Seldon provides a set of python modules to help construct feature pipelines for use inside Seldon.

You can install them in three ways:

## Install from seldon-server project

The module dependencies are:

  * [scikit_learn](http://scikit-learn.org/stable/)
  * [wabbit_wappa](https://github.com/mokelly/wabbit_wappa)
  * [xgboost](https://github.com/dmlc/xgboost)
  * [unicodecsv](https://github.com/jdunck/python-unicodecsv)
  * [kazoo](https://kazoo.readthedocs.org/en/latest/)
  * [boto](https://github.com/boto/boto)

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

**Caveat - at present you will need to install [xgboost](https://github.com/dmlc/xgboost/tree/master/python-package) manually as the pip install fails.**

