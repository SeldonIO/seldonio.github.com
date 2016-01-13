---
layout: default
title: Seldon Shell
---
* [Introduction](#intro)
* [Installing Seldon Shell](#install)
* [Starting the Shell](#startup)
* [Managing Datasources](#db) [ db ]
* [Configuring Memcache](#memcached) [ memcached ]
* [Managing Clients](#client) [ client ]
* [Setting up atrributes](#attr) [ attr ]
* [Importing static data](#import) [ import ]
* [Configuring Recommenders](#alg) [ alg ]
* [Setting up and Running Offline Jobs](#model) [ model ]

## <a name="intro"></a>Introduction

The Seldon Shell is a tool for configuring and managing the Seldon Platform.

It manages zookeeper configuration data for Seldon Server.

It has the usual shell features such command completion and history.

## <a name="install"></a>Installing Seldon Shell

The shell comes bundled with the [Seldon python package](/python-package.html)  
If you using the Seldon virtual machine, it will already be installed.

## <a name="startup"></a>Starting the Shell

The shell can be started by using the following on the command line.

    $ seldon-shell

When the shell is run for the first time, it will create the "seldon.conf" file and exit.

This file can be left with the defaults or modified as appropriate before running the shell again.

To successfully launch the shell it will need to connect to your zookeeper host. This can also be a zookeeper cluster - in which it needs the list of hosts.

There are two ways in which you can ensure it finds the correct zookeeper host.

One way is to modify the "seldon.conf" file and update the "zk_hosts" setting in the json configuration.

{% highlight json %}
{
    ...

    "zk_hosts": "localhost:2181",

    ...
}
{% endhighlight %}

Another way is to use a commandline line override using the --zk-hosts option. This will take precedence over the conf file setting.

    $ seldon-shell --zk-hosts 127.0.0.1


On a successful launch, it will show something similar to the following

    connecting to 127.0.0.1 [SUCCEEDED]
    seldon>

## Seldon Shell Help

At the shell command prompt use the **help** command to get a list of all shell commands.

    seldon> help

Individual commands also have a help sub-command to obtain other sub-commands.

    seldon> db help

## <a name="db"></a>Managing Datasources

The **db** command can be used for configuring MySQL datasources.

Use the following command to check the current settings. Note these may not be committed to zookeeper yet.

    seldon> db

To change any of the settings use

    seldon> db setup <SomeDb>

If the <SomeDb> name is a new datasource - then it will be created. If its an existing one - then it will be updated.

Once the settings are correct, use the following to commit to zookeeper

    seldon> db commit

## <a name="memcached"></a>Configuring Memcache

The **memcached** command can be used for configuring memcached.

Use the following command to check the current settings. Note these may not be committed to zookeeper yet.

    seldon> memcached

To change any of the settings use

    seldon> memcached setup

Once the settings are correct, use the following to commit to zookeeper

    seldon> memcached commit

## <a name="client"></a>Managing Clients

Clients in the Seldon Platform can be considered as a particular datasets that you want to work with.  
The **client** command can be used to setup these datasets.

Use the following to show the list of existing clients

    seldon> client

To create a new client use the following command. It requires an existing datasource that would have been created with the **db** command.

    seldon> client setup <clientName> <dbName>


## <a name="attr"></a>Setting up atrributes

Once a client is setup, the **attr** command can be used to setup the attributes for that data.

Use the following command to edit the attributes for the client. If none have been setup already then a default configuration is generated with some simple attributes such as "title".

    seldon> attr edit <clientName>

To change the editor used for the editing process, update the **EDITOR** environment variable as necessary.

At anytime the following command can be used to show the attributes for the client that have been setup.

    seldon> attr show <clientName>

Once the attributes have been edited, they can be used update the client using the following command.

    seldon> attr apply <clientName>

## <a name="import"></a>Importing static data

The **import** command can used to import static data into your selected client.  
The data should be in csv format that matches the attributes configured for the client.

Use the following set of commands to import items, users and actions.

    seldon> import items <clientName> </path/to/items.csv>
    seldon> import users <clientName> </path/to/users.csv>
    seldon> import actions <clientName> </path/to/actions.csv>

## <a name="alg"></a>Configuring Recommenders

The **alg** command can be used for setting up and changing recommenders.

There is a number of built in recommenders that can be configured for a particular client.
Each client can have one or more recommenders assigned.


To get a list of the available recommenders, use the following command

    seldon> alg


The following command can be used to add a recommender from the available list. The first time this command is used the "recentItemsRecommender" is added by default.
Additional recommenders can also be added this way.

    seldon> alg add <clientName>

The following command can be used to remove recommenders that are not required.

    seldon> alg delete <clientName>

To reorder the recommender list - the following command can be used by picking a recommender to promote. It will be moved up the list.

    seldon> alg promote <clientName>

The following command can be used to check the current list of recommenders for the client. If there are no recommenders setup - this command will add "recentItemsRecommender" by default.

    seldon> alg show <clientName>

To commit the changes to zookeeper, use the following command

    seldon> alg commit <clientName>


## <a name="model"></a>Setting up and Running Offline Jobs

The **model** command can be used for setting up and running offline training jobs.

Use the following command to add a particualar offline job.

    seldon> model add <clientName>

To check which models are already added use the following command.

    seldon> model show <clientName>

Once models are added, its possible to edit their settings using the following command.

    seldon> model edit <clientName>

An offline job for a particular model can be run using the following.

    seldon> model train <clientName>

