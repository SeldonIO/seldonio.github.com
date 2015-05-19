---
layout: default
title: Deploy Steps
---

# Deploy Steps

Steps involved in setting up Seldon from source to serve content recommendations:

* [Start with an understanding of Seldon concepts](/concepts.html)
* [Setup Seldon Server](/seldon-server-setup.html)
* [Configure your data](/item-recommendation-data.html)
* Provide [realtime activity data](/realtime-activity-data.html) 
* [Generate the offline models](/offline-models.html)
* [Algorithm configuration](/config-build-and-deploy.html)
* Start recommending using the [Seldon API](api.html)

Steps involved in setting up Seldon from source to serve predictions (alpha release):

* [Start with an understanding of Seldon concepts](/concepts.html)
* [Setup Seldon Server](/seldon-server-setup.html)
* Injest arbitrary JSON event data via the [REST](api-oauth.html#events) or [Javascript](api-javascript.html) APIs
* Build a model from data
* Create a [pluggable prediction algorithm](pluggable-prediction-algorithms.html)
* [Algorithm configuration](/configuration.html#prediction-algorithms)
* Start predicting using the [Seldon API](api.html)
