---
layout: default
title: Seldon Spark
---

# Introduction
 
The Seldon Spark component is used to run spark jobs that build models to be used by the Seldon Server.

# Set Up

## Prerequisites

To use Seldon Spark, the following need to be installed:

* Java (v = 7)
* Spark (v >= 1.2.0)
* Maven (v >=3)
* Git


## Build the Seldon Spark app jar

        cd ~/
        git clone https://github.com/SeldonIO/seldon-spark
        cd seldon-spark
        git checkout tags/v1.0.1
        mvn -DskipTests=true clean package

# Run spark jobs


