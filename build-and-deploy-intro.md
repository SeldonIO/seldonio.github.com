---
layout: default
title: Introduction to Deploying Seldon 
---

# Introduction to Deploying Seldon

## Open Source Projects
The open source projects that comprise Seldon can be seen in the image below grouped into logical sections. The diagram also illustrate the extension points that can be used to add custom algorithms (offline model creation and real time prediction scoring) to easily extend Seldon.

![Open Source Projects](/img/OpenSourceProjects.png "Open Source Projects (project at https://github.com/SeldonIO")

## External Interfaces

 * [seldon-js-lib](https://github.com/SeldonIO/seldon-server/tree/master/client/js-client) : A javascript library to allow easy integration of Seldon onto existing website
 * [seldon-java-client](https://github.com/SeldonIO/seldon-server/tree/master/client/java-client) : A server to server implementation of the Seldon Oauth REST API in Java
 * [seldon-importer-web](https://github.com/SeldonIO/seldon-server/tree/master/web-item-importer) : Scrape content meta data from web pages and import via Seldon API

## Real-Time Predictive Scoring

 * [seldon-server](https://github.com/SeldonIO/seldon-server) : Real-time predictive scoring that exposes a REST API for external clients.
 * Micro-services for custom algorithms with example python templates for [item recommendation](pluggable-recommendation-algorithms.html#recommender-python-template) and [predictive scoring](pluggable-prediction-algorithms.html#prediction-python-template).

## Offline Modelling

 * [seldon-spark](https://github.com/SeldonIO/seldon-server/tree/master/offline-jobs/spark) : Implementations of predictive models in Spark.
 * Prepackeged Docker integrations with [Vowpal Wabbit](offline-prediction-models.html#vw) and [XGBoost](offline-prediction-models.html#xgboost)
 * [Docker container for Semantic Vectors](semantic-vectors.html) : Vector based models for language modelling using Semantic vectors
