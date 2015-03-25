---
layout: default
title: Deploy Seldon Server
---

# Deploy Seldon Server

## Prerequisites
 
To run the Seldon Server, we need a number of things present on the machine it is running on

* Java (v >= 7)
* Tomcat (v >= 7)
* Maven (v >=3)
* [td-agent](http://www.fluentd.org/download) (latest)

There also needs to be a ZooKeeper server running along with Memcached and a MySQL server. It's up to you where you place these, but they should be accessible from the server on which you are running Seldon Server.

## Config project

The project Seldon [seldon-server-config-template](https://github.com/SeldonIO/seldon-server-config-template) provides a template for the configuration required in the Seldon Server. Clone it and edit `server.properties`, filling in the properties to match your set up. Here is what the properties mean:

* **CLIENT_NAME** - in this initial setup doc we only allow one client. This name should be without spaces and without special characters. It is the name you will use in the REST API etc from now on.
* **ZK_HOST** - the host name of the server that contains the zookeeper instance. This will most likely be localhost.
* **MEMCACHED_HOST** - the host name of the server that contains the Memcached instance. This will most likely be localhost.
* **MEMCACHED_PORT** - see above.
* **DB_HOST** - the host name of the MySQL instance.
* **DB_USER** - the user name of the above.
* **DB_PWD** - the password of the above.
* **LOGS_DIR** - where to output the Seldon Server logs. This can be any directory.
* **TD_LOGS_DIR** - where to output the td-agent logs. These are used for the input for the Spark jobs. It is probable that you are running the Spark jobs on the same machine. If so then this can be any directory. If you are running the Spark jobs on a separate machine then you should give a folder that is on a shared drive.

Run `./rewrite_properties.sh` to propagate the properties from `server.properties` into other property files. Then you are ready to build the config set. Run `mvn clean install` to install the config for use by the Seldon Server. Also produced is the td-agent.conf file that needs to replace the default. The location of this file depends on what platform you are runing on, for me it is `/etc/td-agent/td-agent.conf`. Overwrite this file with the one in the config project and restart td-agent.

## The main project

Clone the project [seldon-server](https://github.com/SeldonIO/seldon-server) and run `mvn clean package`. This will create a war file in the target directory. Deploy this to an Apache Tomcat instance (we recommend that you deploy it at ROOT). You should now have a running Seldon Server instance!



