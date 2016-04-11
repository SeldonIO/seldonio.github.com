---
layout: default
title: Seldon CLI
---
* [Introduction](#intro)
* [Installing Seldon Cli](#install)
* [Using the Seldon Cli](#usingthecli)
* [Managing Datasources](#db)
* [Configuring Memcache](#memcached)
* [Managing Clients](#client)
* [Setting up atrributes](#attr)
* [Importing static data](#import)
* [Configuring Recommenders](#rec_alg)
* [Setting up and Running Offline Modeling Jobs](#model)

## <a name="intro"></a>Introduction

The Seldon Cli is a tool for configuring and managing the Seldon Platform.

It manages zookeeper configuration data for Seldon Server.


## <a name="install"></a>Installing Seldon Cli

The cli tool comes bundled with the [Seldon python package](/python-package.html)  
If using the Seldon virtual machine, it will already be installed.


## <a name="usingthecli"></a>Using the Seldon Cli

To setup the configuration that the cli needs, run the following command:

    $ seldon-cli --setup-config

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

    $ seldon-cli --zk-hosts 127.0.0.1


## <a name="db"></a>Managing Datasources

The **db** command can be used for configuring MySQL datasources.

Use the following command to check the current settings. Note these may not be committed to zookeeper yet.

    $ seldon-cli db --action show

For just a list of the db sources use:

    $ seldon-cli db --action list

To create/update db changes use the following:

    $ seldon-cli db --action setup --db-name <SomeDb> --db-user <User> --db-password <Password> --db-jdbc <JDBC String>

Here is an example:

    $ seldon-cli db --action setup --db-name ClientDB --db-user root --db-password mypass --db-jdbc 'jdbc:mysql:replication://127.0.0.1:3306,127.0.0.1:3306/?characterEncoding=utf8&useServerPrepStmts=true&logger=com.mysql.jdbc.log.StandardLogger&roundRobinLoadBalance=true&transformedBitIsBoolean=true&rewriteBatchedStatements=true'

If its a new datasource - then it will be created. If its an existing one - then it will be updated.

Once the settings are correct, use the following to commit to zookeeper

    $ seldon-cli db --action commit


## <a name="memcached"></a>Configuring Memcache

The **memcached** command can be used for configuring memcached.

Use the following command to check the current settings. Note these may not be committed to zookeeper yet.

    $ seldon-cli memcached

To change any of the settings use the setup action, eg.

    $ seldon-cli memcached --action setup --numClients 4 --servers "localhost:11211"

Once the settings are correct, use the following to commit to zookeeper

    $ seldon-cli memcached --action commit


## <a name="client"></a>Managing Clients

Clients in the Seldon Platform can be considered as particular datasets that you want to work with.  
The **client** command can be used to setup these datasets.

Use the following to show the list of existing clients:

    $ seldon-cli client --action list

To create a new client use the following command. It requires an existing datasource that would have been created with the **db** command.

    $ seldon-cli client --action setup --db-name <dbName> --client-name <clientName>


## <a name="attr"></a>Setting up atrributes

Once a client is setup, the **attr** command can be used to setup the attributes for that data.

Use the following command to edit the attributes for the client. If none have been setup already then a default configuration is generated with some simple attributes such as "title".

    $ seldon attr --action edit --client-name <clientName>

To change the editor used for the editing process, update the **EDITOR** environment variable as necessary.

At anytime the following command can be used to show the attributes for the client that have been setup.

    $ seldon attr --action show --client-name <clientName>

Once the attributes have been edited, they can be used update the client using the following command.

    $ seldon attr --action apply --client-name <clientName>

## <a name="import"></a>Importing static data

The **import** command can used to import static data into your selected client.  
The data should be in csv format that matches the attributes configured for the client.

Use the following set of commands to import items, users and actions.

    $ seldon import items <clientName> </path/to/items.csv>
    $ seldon import users <clientName> </path/to/users.csv>
    $ seldon import actions <clientName> </path/to/actions.csv>


## <a name="rec_alg"></a>Configuring Recommenders

There is a number of built in recommenders that can be configured for a particular client.
Each client can have one or more recommenders assigned.

To list the names of the available recommenders, use the following command

    $ seldon rec_alg --action list

The following command can be used to add a recommender from the available list. The first time this command is used the "recentItemsRecommender" is added by default.
Additional recommenders can also be added this way.

    $ seldon rec_alg --action add --client-name <clientName> --recommender-name <recommenderName>

The following command can be used to remove recommenders that are not required.

    $ seldon rec_alg --action delete --client-name <clientName> --recommender-name <recommenderName>

The following command can be used to check the current list of recommenders for the client. If there are no recommenders setup - this command will add "recentItemsRecommender" by default.

    $ seldon rec_alg --action show --client-name <clientName>

To commit the changes to zookeeper, use the following command

    $ seldon rec_alg --action commit --client-name <clientName>


## <a name="model"></a>Setting up and Running Offline Modeling Jobs

TODO

