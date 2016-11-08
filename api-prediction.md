---
layout: default
title: Seldon Prediction APIs
---

# Seldon Prediction APIs

## Introduction
Seldon provides three variants prediction API:

 * OAUTH REST
 * Javascript
 * gRPC

Authorization for the gRPC API is via an OAUTH token.

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

* [REST API](api-oauth-prediction.html)
* [JS API](api-javascript-prediction.html)
* [gRPC API](grpc.html)

