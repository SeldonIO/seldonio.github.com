---
layout: default
title: Deploy Steps
---

# Deployment Guide
Follow the steps below to set Seldon up to provide recommendation or prediction services.

# Content Recommendation

Steps involved in setting up Seldon from source to serve content recommendations:

 1. [Start with an understanding of Seldon concepts](/concepts.html)
 1. [Setup Seldon Server](/seldon-server-setup.html)
 1. [Configure your data](/item-recommendation-data.html)
 1. Provide [realtime activity data](/realtime-activity-data.html) 
 1. [Generate an offline model](/offline-models.html)
 1. [Algorithm configuration](/runtime-recommendation.html)
 1. Start recommending using the [Seldon API](api.html)

# General Predictive Services

Steps involved in setting up Seldon from source to serve predictions (alpha release):

 1. [Start with an understanding of Seldon concepts](/concepts.html)
 1. [Setup Seldon Server](/seldon-server-setup.html)
 1. Injest arbitrary JSON event data via the [REST](api-oauth.html#events) or [Javascript](api-javascript.html) APIs
 1. [Generate an offline model](offline-prediction-models.html)
 1. [Algorithm configuration](/runtime-prediction.html)
 1. Start predicting using the [Seldon API](api.html)
