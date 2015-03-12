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

An example of running the actions job

        cd ~/seldon-spark

        DATE_YESTERDAY=$(perl -e 'use POSIX;print strftime "%Y%m%d",localtime time-86400;')
        INPUT_DATE_STRING=${DATE_YESTERDAY}
        JAR_FILE_PATH=./target/seldon-spark-1.0.1-jar-with-dependencies.jar
        SPARK_HOME=/opt/spark

        INPUT_DIR=~/seldon-logs
        OUTPUT_DIR=~/seldon-models

        ${SPARK_HOME}/bin/spark-submit \
            --class "io.seldon.spark.actions.GroupActionsJob" \
            --master local[1] \
            ${JAR_FILE_PATH} \
                --aws-access-key-id "" \
                --aws-secret-access-key "" \
                --input-path-pattern "${INPUT_DIR}/fluentd/actions.%y/%m%d/*/*" \
                --input-date-string "${INPUT_DATE_STRING}" \
                --output-path-dir "${OUTPUT_DIR}" \
                --gzip-output

