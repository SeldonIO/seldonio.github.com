---
layout: default
title: Seldon Server Setup
---

# Deploy Seldon Server<a name="deploy-server"></a>

## Prerequisites

To run the Seldon Server, we need a number of things present on the machine it is running on.

* Java (v >= 7)
* Tomcat (v >= 7)
* Maven (v >=3)
* [td-agent](http://www.fluentd.org/download) (latest)

There also needs to be a ZooKeeper server running along with Memcached and a MySQL server. It's up to you where you place these, but they should be accessible from the server on which you are running Seldon Server.

## The main project

Clone the project [seldon-server](https://github.com/SeldonIO/seldon-server).

## Configure

The repository for Seldon Server settings is ZooKeeper, but we have provided a script to populate the base properties and set up the database. This is located in the scripts folder. This script consumes the settings in ```./server_config.json```. You need to specify a few things such the locations of the memcached and MySQL server. Note that you can provide more than one MySQL/memcached server if you require this. You will also need to provide a name for the client and which MySQL DB to associate it with. Once this is done, run ```./scripts/initial_setup.py```. This will store your configuration in zookeeper. For the full set of available configuration options see [here](configuration.html).

## Run

Change directory to `seldon-server/server`.
Run `mvn clean package`.
This will create a war file in the target directory. You can now deploy this to an application server. At startup the seldon-server searches for the enviroment variable `SELDON_ZKSERVERS` to find the location of zookeeper. For Apache Tomcat you an achieve this by placing a file `setenv.sh` in the `/bin` folder of tomcat containing:

```
export SELDON_ZKSERVERS=<zookeeper hosts>
```

for example:

```
export SELDON_ZKSERVERS=localhost
```

You should now have a running Seldon Server instance!





