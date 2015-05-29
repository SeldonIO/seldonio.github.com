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

The Seldon AMI provides the components and functionality necessary to use the Seldon platform.

* Support Services - Zookeeper, Memcached, td-agent and MySQL Server

    These can be all be started and stopped together as follows:

        cd ~/seldon-server/vm/bin/
        ./start-all
        ./stop-all

    Or individually:

        cd ~/seldon-server/vm/bin/
        ./zookeeper start
        ./zookeeper stop
        ./memcached start
        ./memcached stop
        ./td-agent start
        ./td-agent stop
        ./mysql start
        ./mysql stop

* Servlet Container for hosting the Seldon API with Tomcat

    The Seldon API can be started and stopped as follows:

        cd ${TOMCAT_HOME}/bin
        ./startup.sh
        ./shutdown.sh

* Examining and setting data in zookeeper

        cd ~/seldon-server/vm/bin/
        ./zkshell

* Examining and setting data in mysql

        cd ~/seldon-server/vm/bin/
        ./mysql-shell

* Checking the logs

        cd ${TOMCAT_HOME}/logs

* Building and Testing the movie demo

        cd ~/movie-demo-setup
        ./create-movie-demo

    Once the build process finishes, start the Seldon API

        cd ${TOMCAT_HOME}/bin
        ./startup.sh
        
    The demo can be seen with the following url:

        http://localhost:8080/movie-demo/


    The API can be explored using Swagger:

        http://localhost:8080/swagger/

