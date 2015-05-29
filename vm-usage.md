---
layout: default
title: Seldon VM usage
---

## Getting started with the Seldon Virtual Machines

The Seldon Virtual Machines (both Vagrant and AMI) provide the components and functionality necessary to use the Seldon platform.

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

# Movie Recommender Demo
We provide scripts to create a movie recommendaton demo. To set this up see [here](movie-recommender-demo.html).

# Build models with your data
Follow the [deployment steps](deploying-steps.html) to build models using your own data.
