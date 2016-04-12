---
layout: default
title: Seldon CLI
---
* [Introduction](#intro)
* [Prerequisites](#prerequisites)
* [**seldon-cli db**](#db)
* [**seldon-cli memcached**](#memcached)
* [**seldon-cli client**](#client)
* [**seldon-cli keys**](#keys)
* [**seldon-cli attr**](#attr)
* [**seldon-cli import**](#import)
* [**seldon-cli model**](#model)
* [**seldon-cli rec_alg**](#rec_alg)
* [**seldon-cli predict_alg**](#predict_alg)
* [**seldon-cli api**](#api)
* [Manual install outside Kubernetes](#manual)

# <a name="intro"></a>**Introduction**

The Seldon CLI is a tool for configuring and managing the Seldon Platform.

# **Prerequisites**<a name="prerequisites"></a>

 * [You have installed Seldon on a Kubernetes cluster](install.html)
 * You haved added ```seldon-server/kubernetes/bin``` to you shell PATH environment variable.


# <a name="db"></a>**seldon-cli db**

## Synopsis
Create and configure MySQL datasources.

{% highlight bash %}
seldon-cli db --action ACTION --db-name DB_NAME --db-user USER --db-password PASSWORD --db-jdbc JDBC_STRING
{% endhighlight %}

## Examples

{% highlight bash %}
# Use the following command to check the current settings. Note these may not be committed to zookeeper yet.
seldon-cli db --action show
{% endhighlight %}

{% highlight bash %}
# For just a list of the db sources use
seldon-cli db --action list
{% endhighlight %}

{% highlight bash %}
# To create/update db changes use the following.
# If its a new datasource - then it will be created. If its an existing one - then it will be updated.
seldon-cli db --action setup --db-name ClientDB --db-user root --db-password mypass --db-jdbc 'jdbc:mysql:replication://127.0.0.1:3306,127.0.0.1:3306/?characterEncoding=utf8&useServerPrepStmts=true&logger=com.mysql.jdbc.log.StandardLogger&roundRobinLoadBalance=true&transformedBitIsBoolean=true&rewriteBatchedStatements=true'
{% endhighlight %}

{% highlight bash %}
# Once the settings are correct, use the following to commit to zookeeper
seldon-cli db --action commit
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli db [-h] [--action {show,list,setup,commit}]
                     [--db-name DB_NAME] [--db-user DB_USER]
                     [--db-password DB_PASSWORD] [--db-jdbc DB_JDBC]
                     ...

Seldon Cli

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {show,list,setup,commit}
                        the action to use
  --db-name DB_NAME     the name of the db
  --db-user DB_USER     the user
  --db-password DB_PASSWORD
                        the password for the user
  --db-jdbc DB_JDBC     the jdbc string
{% endhighlight %}


# <a name="memcached"></a>**seldon-cli memcached**

## Synopsis
Configure memcached.

{% highlight bash %}
seldon-cli memcached --action ACTION --numClients NUM_CLIENTS --servers SERVERS_LIST
{% endhighlight %}

## Examples

{% highlight bash %}
# Use the following command to check the current settings. Note these may not be committed to zookeeper yet.
seldon-cli memcached
{% endhighlight %}

{% highlight bash %}
# To change any of the settings use the setup action
seldon-cli memcached --action setup --numClients 4 --servers "localhost:11211"
{% endhighlight %}

{% highlight bash %}
# Once the settings are correct, use the following to commit to zookeeper
seldon-cli memcached --action commit
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli memcached [-h] [--action {setup,commit}]
                            [--numClients NUMCLIENTS] [--servers SERVERS]
                            ...

Seldon Cli

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {setup,commit}
                        the action to use
  --numClients NUMCLIENTS
                        number of clients
  --servers SERVERS     the server list
{% endhighlight %}


# <a name="client"></a>**seldon-cli client**

## Synopsis

Clients in the Seldon Platform can be considered as particular datasets that you want to work with.  
The client command can be used to setup these datasets.

{% highlight bash %}
seldon-cli client --action ACTION --db-name DB_NAME --client-name CLIENT_NAME --input-date-string INPUT_DATE
{% endhighlight %}

## Examples

{% highlight bash %}
# Use the following to show the list of existing clients
seldon-cli client --action list
{% endhighlight %}

{% highlight bash %}
# To create a new client use the following command. It requires an existing datasource that would have been created with the db command.
seldon-cli client --action setup --db-name ClientDB --client-name testclient
{% endhighlight %}

{% highlight bash %}
# Run a job to collect actions data for a particular day (in YYYYMMDD format), for one client
seldon-cli client --action processactions --client-name testclient --input-date-string 20160216

# Run a job to collect actions data for a particular day (in YYYYMMDD format), for all clients
seldon-cli client --action processactions --input-date-string 20160216
{% endhighlight %}

{% highlight bash %}
# Run a job to collect events data for a particular day (in YYYYMMDD format), for one client
seldon-cli client --action processevents --client-name mytest1 --input-date-string 20160216

# Run a job to collect events data for a particular day (in YYYYMMDD format), for all clients
seldon-cli client --action processevents --input-date-string 20160216
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli client [-h]
                         [--action {list,setup,processactions,processevents}]
                         [--db-name DB_NAME] [--client-name CLIENT_NAME]
                         [--input-date-string INPUT_DATE_STRING]
                         ...

Seldon Cli

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {list,setup,processactions,processevents}
                        the action to use
  --db-name DB_NAME     the name of the db
  --client-name CLIENT_NAME
                        the name of the client
  --input-date-string INPUT_DATE_STRING
                        The date to process in YYYYMMDD format
{% endhighlight %}

# <a name="keys"></a>**seldon-cli keys**
Display Oauth or JS authenetication keys for a client.

## Synopsis
Show the keys for a client, either every key or just js or oauth keys.

{% highlight bash %}
seldon-cli keys --action list --client-name CLIENT --scope SCOPE {js|all}
{% endhighlight %}

## Examples

{% highlight bash %}
# Show keys for client test
seldon-cli keys --client-name test
{% endhighlight %}

{% highlight bash %}
# Show keys for client test
# show js key for client test in quiet mode
seldon-cli --quiet keys --client-name test --scope js
{% endhighlight %}



## Options

{% highlight bash %}
usage: seldon-cli keys [-h] [--action {list}] [--client-name CLIENT_NAME]
                       [--scope {js,all}]
                       ...

Seldon CLI

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {list}       the action to use
  --client-name CLIENT_NAME
                        the name of the client
  --scope {js,all}      the key scope
{% endhighlight %}




# <a name="attr"></a>**seldon-cli attr**

## Synopsis

Once a client is setup, the attr command can be used to setup the attributes for that data.

{% highlight bash %}
seldon-cli attr --action ACTION --client-name CLIENT_NAME
{% endhighlight %}

## Examples

{% highlight bash %}
# Use the following command to edit the attributes for the client. If none have been setup already then a default configuration is generated with some simple attributes such as "title".
# To change the editor used for the editing process, update the EDITOR environment variable as necessary.
seldon-cli attr --action edit --client-name testclient
{% endhighlight %}

{% highlight bash %}
# At anytime the following command can be used to show the attributes for the client that have been setup.
seldon-cli attr --action show --client-name testclient
{% endhighlight %}

{% highlight bash %}
# Once the attributes have been edited, they can be used update the client using the following command.
seldon-cli attr --action apply --client-name testclient
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli attr [-h] [--action {edit,show,apply}]
                       [--client-name CLIENT_NAME] [--json JSON]
                       ...

Seldon Cli

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {edit,show,apply}
                        the action to use
  --client-name CLIENT_NAME
                        the name of the client
  --json JSON           the file containing attr json
{% endhighlight %}


# <a name="import"></a>**seldon-cli import**

## Synopsis

The import command can used to import static data into your selected client.  
The data should be in csv format that matches the attributes configured for the client.

{% highlight bash %}
seldon-cli import --action ACTION --client-name CLIENT_NAME --file-path PATH_TO_FILE
{% endhighlight %}


## Examples

{% highlight bash %}
# Use the following set of commands to import items, users and actions.
seldon-cli import --action items --client-name testclient --file-path /path/to/items.csv
seldon-cli import --action users --client-name testclient --file-path /path/to/users.csv
seldon-cli import --action actions --client-name testclient --file-path /path/to/actions.csv
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli import [-h] --action {items,users,actions} --client-name
                         CLIENT_NAME --file-path FILE_PATH
                         ...

Seldon Cli

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {items,users,actions}
                        the action to use
  --client-name CLIENT_NAME
                        the name of the client
  --file-path FILE_PATH
                        path to the data file
{% endhighlight %}



# <a name="model"></a>**seldon-cli model**

## Synopsis

The model command can be used for setting up and running offline training jobs.

{% highlight bash %}
seldon-cli model --action ACTION --client-name CLIENT_NAME --model-name MODEL_NAME
{% endhighlight %}

## Examples

{% highlight bash %}
# Use the following command to get a list of names for the available models
seldon-cli model --action list
{% endhighlight %}

{% highlight bash %}
# Use the following command to add a particualar offline job.
seldon-cli model --action add --client-name testclient --model-name matrix-factorization
{% endhighlight %}

{% highlight bash %}
# To check which models are already added use the following command.
seldon-cli model --action show --client-name testclient
{% endhighlight %}

{% highlight bash %}
# Once models are added, its possible to edit their settings using the following command.
seldon-cli model --action edit --client-name testclient --model-name matrix-factorization
{% endhighlight %}

{% highlight bash %}
# An offline job for a particular model can be run using the following.
seldon-cli model --action train --client-name testclient --model-name matrix-factorization
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli model [-h] [--action {list,add,show,edit,train}]
                        [--client-name CLIENT_NAME] [--model-name MODEL_NAME]
                        ...

Seldon Cli

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {list,add,show,edit,train}
                        the action to use
  --client-name CLIENT_NAME
                        the name of the client
  --model-name MODEL_NAME
                        the name of the client
{% endhighlight %}



# <a name="rec_alg"></a>**seldon-cli rec_alg**

## Synopsis

There is a number of built in recommenders that can be configured for a particular client.
Each client can have one or more recommenders assigned.

## Examples

{% highlight bash %}
# List the names of the available recommenders
seldon-cli rec_alg --action list
{% endhighlight %}

{% highlight bash %}
# The following command can be used to add a recommender from the available list. The first time this command is used the "recentItemsRecommender" is added by default.
# Additional recommenders can also be added this way.
seldon-cli rec_alg --action add --client-name testclient --recommender-name recentMfRecommender
{% endhighlight %}

{% highlight bash %}
# The following command can be used to remove recommenders that are not required.
seldon-cli rec_alg --action delete --client-name testclient --recommender-name recentMfRecommender
{% endhighlight %}

{% highlight bash %}
# The following command can be used to check the current list of recommenders for the client.
# If there are no recommenders setup - this command will add "recentItemsRecommender" by default.
seldon-cli rec_alg --action show --client-name testclient
{% endhighlight %}

{% highlight bash %}
# Commit the changes to zookeeper
seldon-cli rec_alg --action commit --client-name testclient
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli rec_alg [-h] --action {list,add,delete,show,commit}
                          [--alg-type {recommendation,prediction}]
                          [--client-name CLIENT_NAME]
                          [--recommender-name RECOMMENDER_NAME]
                          [--config CONFIG]
                          ...

Seldon Cli

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {list,add,delete,show,commit}
                        the action to use
  --alg-type {recommendation,prediction}
                        type of algorithm
  --client-name CLIENT_NAME
                        the name of the client
  --recommender-name RECOMMENDER_NAME
                        the name of recommender
  --config CONFIG       algorithm specific config in the form x=y
{% endhighlight %}



# <a name="predict_alg"></a>**seldon-cli predict_alg**
Configure the Seldon server runtime scoring algorithms for prediction. 

## Synopsis
Configure the server for prediction runtime scorer for a client. At present this is restricted to externalPredictionServer which is an internal microservice.

{% highlight bash %}
seldon-cli predict_alg  --action ACTION --client-name CLIENT --predictor-name PREDICTOR-NAME
{% endhighlight %}

## Examples

{% highlight bash %}
# add a microserice endpoint for the client iris running at given url and having given name
seldon-cli predict_alg  --action add --client-name iris --predictor-name externalPredictionServer --config io.seldon.algorithm.external.url=http://iris-example:5000/predict --config io.seldon.algorithm.external.name=iris-demo
{% endhighlight %}

## Options

{% highlight bash %}
usage: seldon-cli predict_alg [-h] --action {list,show,add,delete,commit}
                              [--client-name CLIENT_NAME]
                              [--predictor-name PREDICTOR_NAME]
                              [--config CONFIG]
                              ...

Seldon CLI

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {list,show,add,delete,commit}
                        the action to use
  --client-name CLIENT_NAME
                        the name of the client
  --predictor-name PREDICTOR_NAME
                        the name of predictor
  --config CONFIG       algorithm specific config in the form x=y
{% endhighlight %}

# <a name="api"></a>**seldon-cli api**
You can call the Seldon API via the Seldon CLI to test the various endpoints.

## Synopsis
Call the javascript or OAuth APIs for a particular client. Specify the endpoint and attributes apparopriate for the endpoint.

{% highlight bash %}
seldon-cli api --client-name CLIENT --endpoint ENDPOINT
{% endhighlight %}

## Examples

### Content Recommendation
Some example content recommendation examples are shown below:

{% highlight bash %}
# Send a new action using javascript API for user 22 for item 10 for client movielnes
seldon-cli api --client-name ml100k --endpoint /js/action/new --user 22 --item 10
{% endhighlight %}


{% highlight bash %}
# Send a new action using OAuth API for user 22 for item 10 for client reuters
seldon-cli api --client-name ml100k --endpoint "/actions" --user 23 --item 10
{% endhighlight %}

{% highlight bash %}
# Get recommendations using the javascript API for user 1 passing item 50 as the current item they are interacting with
seldon-cli api --client-name ml100k --endpoint /js/recommendations --user 1 --item 50 --limit 5
{% endhighlight %}

{% highlight bash %}
# Get recommendations using oauth API for the reuters client for user 1
seldon-cli api --client-name reuters --endpoint "/users/recommendations" --user 1 
{% endhighlight %}

### General Prediction

{% highlight bash %}
# Send a new event to client test with the given JSON using the javascipt API
seldon-cli  api --client-name test --endpoint "/js/event/new" --json '{"f1":1,"f2":2.7,"f3":5.3,"f4":1.9,"target":1}'
{% endhighlight %}

{% highlight bash %}
# Get predictions for client test sending the given JSON
seldon-cli  api --client-name test --endpoint /js/predict --json '{"f1":1,"f2":2.7,"f3":5.3,"f4":1.9}'
{% endhighlight %}

### Options

{% highlight bash %}
usage: seldon-cli api [-h] [--action {call}] --client-name CLIENT_NAME
                      --endpoint
                      {/js/action/new,/js/recommendations,/js/predict,/js/event/new,/actions,/users/recommendations,/predict,/events,/items}
                      [--method {GET,POST}] [--user USER] [--item ITEM]
                      [--type TYPE] [--limit LIMIT] [--json JSON]
                      [--dimensions DIMENSIONS] [--full FULL]
                      [--attributes ATTRIBUTES]
                      ...

Seldon CLI

positional arguments:
  args

optional arguments:
  -h, --help            show this help message and exit
  --action {call}       the action to use
  --client-name CLIENT_NAME
                        the name of the client
  --endpoint {/js/action/new,/js/recommendations,/js/predict,/js/event/new,/actions,/users/recommendations,/predict,/events,/items}
                        api to use
  --method {GET,POST}   http method
  --user USER           user
  --item ITEM           item
  --type TYPE           type
  --limit LIMIT         limit
  --json JSON           json
  --dimensions DIMENSIONS
                        dimensions
  --full FULL           whether to return full attributes true|false
  --attributes ATTRIBUTES
                        attributes

{% endhighlight %}


# <a name="manual"></a>**Manual install outside Kubernetes**

You can ignore this section if you are running Seldon inside Kubernetes.

The seldon-cli program comes packaged in the [Seldon python package](python-package.html). If you want to do a custom install of Seldon outside Kubernetes you will need to install this package. 

To setup the configuration that the cli needs, run the following command:

{% highlight bash %}
seldon-cli --setup-config
{% endhighlight %}

This will create the "seldon.conf" file and indicate where its located.  
The "seldon.conf" will contain default values that can be changed as needed.

To successfully use the cli, it will need to connect to a zookeeper host. This can also be a zookeeper cluster - in which it needs the list of hosts.

There are two ways in which the cli can find the correct zookeeper host.

One way is to modify the "seldon.conf" file and update the "zk_hosts" setting in the json configuration.

{% highlight json %}
{
    ...

    "zk_hosts": "localhost:2181",

    ...
}
{% endhighlight %}

Another way is to use a commandline line override using the --zk-hosts option. This will take precedence over the conf file setting.

{% highlight bash %}
seldon-cli --zk-hosts 127.0.0.1
{% endhighlight %}


