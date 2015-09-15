---
layout: default
title: Seldon APIs
---

##### Content Recommendation Steps

[concepts](/concepts.html) --> [setup server](/seldon-server-setup.html) --> [logging](/seldon-logging.html) --> [configure data](/item-recommendation-data.html) --> [realtime activity data](/realtime-activity-data.html) --> [offline model](/offline-models.html) --> [runtime configuration](/runtime-recommendation.html) --> [microservices](pluggable-recommendation-algorithms.html) --> **recommendations**


# Seldon Content Recommendation API

## Introduction
This document provides an overview of how to use the Seldon OAuth [REST API](api-oauth.html) and [JavaScript API](api-javascript.html), including the available methods, the inputs required for personalized recommendations and the output specifications. Seldon provides a REST API with JSON for data representation and OAuth 2.0 for authorisation. For further clarification on any section please contact support at seldon dot io.

## OAuth Authorisation

A service will be granted

* a consumer_key k
* a consumer_secret s

An authorisation_token t can be retrieved with this API call
{% highlight http %}
POST     /token?consumer_key=k&consumer_secret=s
{% endhighlight %}

## JavaScript Authorization

A service will be provided with a consumer_key which should be passed with each JavaScript request.


# Content Recommendation API Details

* [JS API](api-javascript.html)
* [REST API](api-oauth.html)

