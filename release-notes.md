---
layout: default
title: Release Notes
---

# v1.4.5

 * Google Cloud ingress to allow TLS security over external IP to Seldon API on Google Container Platform
 * Updated Kafka-stream analytics using 0.10.2.1 of kafka/kafka-streams and running as Kubernetes Deployments

# v1.4.4

  * [Kubernetes persistent volumes](http://docs.seldon.io/install.html#storage)  are abstracted by using Persistent Volume Claims. Fixes issue #50.
  * All components run as Kubernetes Deployments. Fixes issue #49. 
  * Initial beta ability to run [external Google MySQL](http://docs.seldon.io/install.html#mysql)  server outside Kubernetes.
  * [Updated luigi configuration](http://docs.seldon.io/content-recommendation-guide.html#model)  for describing ML pipelines and built-in algorithms.

# v1.4.3

 *  Bug fix release for kubernetes configuration 

# v1.4.2

  * Maintenance release.
  * Docs for anomaly detection demo updated and python libraries in docker fixed.

# v1.4.1

  * Maintenance release with updates to [built-in recommendation algorithms](http://docs.seldon.io/content-recommendation-models.html).
  * Initial code for [Anomaly Detection](https://github.com/SeldonIO/seldon-server/blob/master/python/seldon/anomaly/AnomalyDetection.py).

# v1.4

  * This release contains a gRPC interface for prediction calls to complement the REST APIs. Details can be found [here](http://docs.seldon.io/grpc.html) along with a [benchmarking test](http://docs.seldon.io/benchmark-prediction.html).
  * This release contains a breaking change in the prediction [REST](http://docs.seldon.io/api-oauth-prediction.html) and [JS](http://docs.seldon.io/api-javascript-prediction.html) API calls for the format to pass in data.
  * There are also changes to the [seldon scripts](http://docs.seldon.io/scripts.html) including a new script to start REST and RPC microservices and one to start locust load tests.

For the full set of releases see [here](https://github.com/SeldonIO/seldon-server/releases)


