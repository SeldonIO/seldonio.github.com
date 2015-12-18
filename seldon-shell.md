---
layout: default
title: Seldon Shell
---
* [Introduction](#intro)
* [Installing Seldon Shell](#install)
* [Starting the Shell](#startup)
* [Managing Datasources](#db) [ db ]
* [Configuring Memcache](#memcached) [ memcached ]
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

    $ seldon

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

    $ seldon --zk-hosts 127.0.0.1


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

## <a name="memcached"></a>Configuring Memcache

The **memcached** command can be used for configuring memcached.

Use the following command to check the current settings. Note these may not be committed to zookeeper yet.

    seldon> memcached

To change any of the settings use

    seldon> memcached setup

Once the settings are correct, use the following to commit to zookeeper

    seldon> memcached commit

##Managing Clients

**client**

##Setting up atrributes

**attr**

##Importing static data

**import**

## Configuring Recommenders

**alg**

## <a name="model"></a>Setting up and Running Offline Jobs

**model**

