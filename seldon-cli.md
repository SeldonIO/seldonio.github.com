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
* [Configuring Recommenders](#alg)
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

TODO

## <a name="import"></a>Importing static data

TODO

## <a name="alg"></a>Configuring Recommenders

TODO

## <a name="model"></a>Setting up and Running Offline Modeling Jobs

TODO

