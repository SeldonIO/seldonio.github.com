---
layout: default
title: AWS AMI for VM
---

For those with an Amazon Web Services account we provide an all-in-one packaged Seldon as an AMI.

The AMI should be run with at least an m3.medium sized instance.

To get the link please [sign up for the beta program](http://www.seldon.io/open-source)

1. ssh into the running AMI EC2 instance.

1. Start the Seldon services.

        /home/ubuntu/seldon/dist/start-all

1. Use a browser to check that the api has finished initialization and is ready.

        http://127.0.0.1:8080/

1. Explore the API using Swagger.

        http://127.0.0.1:8080/swagger/

1. Next Steps - create a [Movie Recommender Demo](movie-recommender-demo.html)


