---
layout: default
title: Seldon AMI for Amazon Web Services
---

# Seldon AMI for Amazon Web Services

For AWS users, we have created an all-in-one packaged Seldon as a private AMI. 

The Seldon AMI is a self-contained environment with the full Seldon platform pre-configured for you to test with your service data. It contains the same tools as [Seldon VM](vm.html), and a movie recommender demo that serves as an example to show you [how to load your data](data.html).

You should run the Seldon AMI on at least an m3.medium sized instance.

## How to request access
**The Seldon AMI currently has private permissions, so please [request access using this form](https://docs.google.com/forms/d/1a81is8xqZxPNT1ZZMupiD70Y2KvPQbrnYQM6w1RYsBI/viewform) and we will whitelist your AWS account.**

To be the first to receive updates about the AMI, please [sign up for the beta program](http://www.seldon.io/open-source)

## Getting started with the Seldon AMI

1. ssh into the running AMI EC2 instance.

1. Start the Seldon services.

        /home/ubuntu/seldon/dist/start-all

1. Use a browser to check that the API has finished initialization and is ready.

        http://127.0.0.1:8080/

1. Explore the API using Swagger.

        http://127.0.0.1:8080/swagger/

1. Next Steps - create a [Movie Recommender Demo](movie-recommender-demo.html)


