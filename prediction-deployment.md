---
layout: default
title: Prediction Deploy
---

# General Predictive Services

Steps involved in setting up Seldon from source to serve predictions (alpha release):

 1. [Setup Seldon Server](/seldon-server-setup.html) (*not needed if running inside VM*)
 1. Injest arbitrary JSON event data via the [REST](api-oauth-prediction.html#events) or [Javascript](api-javascript-prediction.html) APIs
 1. [Create a feature extraction pipeline](feature-pipeline.html)
 1. [Generate an offline model](offline-prediction-models.html)
 1. [Create a runtime predictive scorer](/runtime-prediction.html) and an optional [microservice predictive scorer](/pluggable-prediction-algorithms.html)
 1. Start predicting using the via the [REST](api-oauth-prediction.html#events) or [Javascript](api-javascript-prediction.html) APIs

# API Reference Docs

 * [Seldon python modules](/python/index.html)
 * [Seldon Prediction API](/api-prediction.html)
