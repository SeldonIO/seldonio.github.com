---
layout: default
title: Analytics
---

# Real Time Analytics

Seldon provides a Grafana dashboard for each client showing real-time analytics. Depending on how you created the seldon kubermetes conf this will be exposed as a NodePort (port 30002) on the cluster or as a LoadBalancer.

The dashboards currently show:

 * API Request Time by REST endpoint
 * API Request count by REST endpoint
 * Content Recommendation Stats
   * Overall Impression and Click count along with CTR
   * Impressions and Clicks by rectag and variation. 
   * CTR by rectag and variation. You would use this to monitor running A/B tests.
 * Prediction Stats
   * Counts for each predicted class

An example dashboard is show below:

![Seldon Analytics Dashboard](/img/grafana.png)

An example dashboard showing prediction stats for the Iris demo running is show below:

![Predictive Stats](/img/grafana-prediction.png)

## Internals

The dashboards are created using Spark streaming jobs which read data from Kafka as it is pushed by fluentd from the front end servers. The data is processed and sent to influxb for use by a Grafana frontend. The dashboards, and influxdb data are held in persistent storage so will be available outside the kubernetes cluster for backup.

