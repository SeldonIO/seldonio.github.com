---
layout: default
title: Content Recommendation Technology Overview
---

# Content Recommendation Technology Overview

Seldon content recommendation is made up of many components that work together to deliver the best recommendations. Roughly, all user actions are captured via the REST API and streamed to logs. Those logs are processed in batch and new user models are delivered to the API Server. Then recommendations are delivered via the REST API.

The logical structure is shown below.

![Seldon Logical Diagram](/img/logical.png)

* External systems interact with Seldon via the REST API. Either via the [JS implementation](api-javascript.html) or via the full [Oauth implementation](api-oauth.html). 
* The core Content Recommendation process interacts with Storage to get models and item meta-data needed to provide recommendations.
* As the API registers activity Real-time Processing may update models and parameters.
* Offline Processing will periodically update the core models and do other admin tasks.
* Client Configuration will allow algorithms to be changed and A/B tests to be run with no down time.
* Real-time Stats are generated to allow effective monitoring
* Article Importers may periodically run to scrape content from web sites to update item meta data

Seldon operates as a set of inter-connected components. Seldon provides a full set of components to create a fully working systems as shown below. However, users are free to swap in their own infrastructure components that subsumes some subset of these, such as using their own JDBC compliant database engine.

![Seldon infrastructure Diagram](/img/infrastructure.png)

The Seldon infrastructure is best described as a set of layers.

* *Real-Time Layer* : responsible for handling the live predictive API requests.
* *Storage Layer* : various types of storage used by other components.
* *Near time / Offline Layer* : components that run compute intensive or otherwise non-realtime jobs.
* *Stats layer* : components to monitor and analyze the running system.

## Real-time Layer

* The API server handles the Seldon [REST](api-oauth.html) and [JavaScript](api-javascript.html) APIs. It is a state-less server than can be replicated to handle the required scale. 
* Zookeeper and Consul servers provide real-time configuration allow new models to be placed into production and client settings to be modified with no down time in the API.
* Fluentd handles the logging of events from the API server and passing these on to storage (e.g., S3) and real-time processing via Kafka.

## Storage Layer

* JDBC database. Used mainly to store user and item meta data.
* A file system of some kind to hold activity data (actions) and the models derived from that data.
* Memcache cluster to provide a caching layer for the API servers to help with load.

## Near Time / Offline Layer

* A streaming Spark cluster used to update models and statistics about a running system, e.g., the most popular items from the last hour.
* An offline Spark cluster which runs various model creation tasks at periodic intervals.
* Various single server tasks. Some examples are:
Model creation pipelines using Vowpal wabbit or Semantic Vectors. 
Scraping of content from web sites to capture item meta data for newly published items.

## Stats Layer (not yet provided)

* Influxdb is used to store real-time stats.
* Grafana dashboards.



