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
 * Micro-services for custom algorithms : [Pluggable custom algorithms](http://docs.seldon.io/external-algorithms.html) with [example python template](https://github.com/SeldonIO/seldon-server/tree/master/external-recommender/python) 

## Offline Modelling

 * [seldon-spark](https://github.com/SeldonIO/seldon-server/tree/master/offline-jobs/spark) : Implementations of predictive models in Spark.
 * [semantic-vectors-lucene-tools](https://github.com/SeldonIO/semantic-vectors-lucene-tools) : Vector based models for language modelling using Semantic vectors
 * Custom Offline Models : Seldon is not prescriptive in how models are created. They simply need a runtime scoring component to serve their predictions.

## Demo/Evaluation

Please join our [beta list](http://www.seldon.io/open-source) to get easy access to these demos.

 * [seldon-vm](https://github.com/SeldonIO/seldon-vm) : An all-one virtual machine with all Seldon components packaged and ready to run. Provides a movie recommendation demo built from open source data.
 * AWS AMI : An AWS AMI with Seldon on a single machine EC2 instance.
