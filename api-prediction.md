---
layout: default
title: Seldon Prediction APIs
---

# Seldon Prediction APIs

## Introduction
This document provides an overview of how to use the Seldon OAuth [REST API](api-oauth.html) and [JavaScript API](api-javascript.html), including the available methods, the inputs required for injesting raw data and the output specifications for predictions on that data. Seldon provides a REST API with JSON for data representation and OAuth 2.0 for authorisation. For further clarification on any section please contact support at seldon dot io.

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


# Prediction API Details

* [JS API](api-javascript-prediction.html)
* [REST API](api-oauth-prediction.html)

