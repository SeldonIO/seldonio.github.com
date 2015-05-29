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

