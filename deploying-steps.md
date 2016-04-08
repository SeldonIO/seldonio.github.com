---
layout: default
title: Deploy Steps
---

# Content Recommendation 
This guide takes you through the steps to set up Seldon to serve content recommendation.


# Import Meta-data

# Collect activity data

# Create a Recommendation Model

## Built-in Spark Model

## Microservice

# Configure runtime recommendation scoring

# Serve recommendations

[Seldon Content Recommedation API](/api.html)

# Advanced Setting

##  Run A/B Tests
When running multiple recommendation models in production you will want to A/B new models to check they perform better than existing models with live clients before you place them fully into production for all users.

The ability to run A/B and multivariant tests is available within Seldon but not yet exposed via the Seldon CLI. Internally everything is controlled via Zookeeper settings. For those with an understanding of Zookeeper who wish to activiate this functionality can find the details [here](advanced-recommender-config.html#multivariate-tests)

## Combine Multiple Algorithms

In some setting you may wish to combine multipe algorithms together to get a combined result. Currently this is not available in the seldon CLI but those with an understanding of Zookeeper can find the details [here](advanced-recommender-config.html#cascading-algorithms)

## API Controlled Variants

In some more complex content recommendation installation you may want to control for a particualar single client (single API key) various different algorithms for different settings, e.g. provide in-section Sport content recommendations and cross-site general content recommendations in different parts of a web page. The changes need to implement this are discussed [here](advanced-recommender-config.html#recommendation-variants)




