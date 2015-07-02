---
layout: default
title: Seldon Logging
---

# Seldon Logging

The Seldon server generates logs on disk in response to handling requests on the rest api. Some of the logs are used for monitoring the inner workings of the server, whilst others are used as a source of data for further processing.

##Actions Logging

Actions are logged by the server in response to processing calls received on the following endpoints:

{% highlight text %}
/js/action/new
/actions
{% endhighlight %}

These generate entries in the following log:

{% highlight bash %}
${TOMCAT_HOME}/logs/seldon-server/actions/actions.log
{% endhighlight %}

The program [td-agent](http://docs.treasuredata.com/articles/td-agent) is used to process these log entries. The way it works is essentially to monitor a set of logs and then process the entries in those logs. The rules that determine what gets processed and how, is described in a conf file for td-agent. There is an example of a conf file in the seldon-server project as follows:

{% highlight bash %}
seldon-server/td-agent.conf
{% endhighlight %}

The processed action log entries are are saved in the following location:

{% highlight bash %}
${TOMCAT_HOME}/logs/fluentd/actions.YYYY
{% endhighlight %}

The processed action logs are converted to json data and stored by day and hour. Also they are compressed to save space. An example of one these logs is as follows:

{% highlight bash %}
${TOMCAT_HOME}/logs/fluentd/actions.2015/0630/10/20150630-10_0.log.gz
{% endhighlight %}

The contents of these logs are used by the [Actions Grouping](/spark-models.html#actions) spark job.

