---
layout: default
title: Install
---

# Installing Seldon

 * [Clone Seldon](#clone)
 * [Install Kubernetes](#install-kubernetes)
 * [Create Kubernetes Configuration](#configure)
     * [Persistent Storage](#storage)
     * [Seldon API Endpoint](#endpoint)
 * [Launch Seldon](#launch)
 * [Troubleshooting](#troubleshooting)

# Download Seldon<a name="clone"></a>

{% highlight bash %}
git clone https://github.com/seldonio/seldon-server
{% endhighlight %}

  * add ```seldon-server/kubernetes/bin``` to you shell PATH environment variable.

# Create a Kubernetes Cluster<a name="install-kubernetes"></a>

Seldon runs inside a [Kubernetes](http://kubernetes.io) cluster so you need to follow their [guides](http://kubernetes.io/docs) to create a cluster locally, on servers or in the cloud.

   * Add kubectl to your shell PATH environemnt variable.

# Create Kubernetes Configuration<a name="configure"></a>

Once you have a Kubernetes cluster Seldon can be started as a series of containers which run within it. As a first step you to to create the required JSON Kubernetes files. A Makefile to create these can be found in ```kubernetes/conf```

## Persistent Storage<a name="storage"></a>
Seldon uses a Kubernetes [volume](http://kubernetes.io/docs/user-guide/volumes/) to store and share data between containers. The Makefile allows you to create the Kuberenetes configuration with HostPath and GlusterFS example persistent volumes. You can modify it to use other possible volumes, such as NFS, as allowed by kubernetes.

### HostPath
To create the default HostPath kubernetes conf files set for /seldon-date do the following:

{% highlight bash %}
 cd kubernetes/conf
 make clean conf
{% endhighlight %}

   **Note** : HostPath only makes sense for demo/testing where you have a Kubernetes cluster with a single minion where all containers can share the location on the host. You will need to create the host path folder on your single kubernetes minion.

### GlusterFS
GlusterFS works well for a production setting. For this you will need to have setup your own GlusterFS cluster. The Makefile assumes there is a vaolume called gs0. You will need to provide two ip addresses of two nodes in your GlusterFS cluster.

{% highlight bash %}
 cd kubernetes/conf
 make clean conf GLUSTERFS_IP1=1.2.3.4 GLUSTERFS_IP2=1.2.3.5
{% endhighlight %}

## Seldon API Endpoint<a name="endpoint"></a>
By default the Seldon API server endpoint is set to a Kubernetes NodePort at port 30000. If you run in the cloud you can change this to LoadBalancer, e.g.

{% highlight bash %}
 cd kubernetes/conf
 make clean conf SELDON_SERVICE_TYPE=LoadBalancer
{% endhighlight %}



# Launch Seldon<a name="launch"></a>
Scripts ```seldon-up.sh``` and ```seldon-down.sh``` in ```kubernetes/bin``` start and stop Seldon and should be in your PATH.


To launch seldon run
{% highlight bash %}
seldon-up.sh
{% endhighlight %}

To start with GlusterFS run 
{% highlight bash %}
SELDON_WITH_GLUSTERFS=true seldon-up.sh
{% endhighlight %}

To shutdown seldon run
{% highlight bash %}
seldon-down.sh
{% endhighlight %}

The first time you run seldon-up it may take some time to complete as it will need to download all the images from DockerHub.

# Troubleshooting<a name="troubleshooting"></a>

 * When running ```seldon-up.sh``` the script waits for ever for all pods to be in running state.

If you plan to test Seldon on a non-local cluster you will need to ensure your cluster is large enough to run all the Seldon services or disable the **LimitRanger** plugin. In the current version of Kubernetes to disable this plugin do the following. Edit ```<kubernetes>/cluster/<provider>/config-default.sh``` and remove LimitRanger from the following line:
{% highlight bash %}
ADMISSION_CONTROL=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota,PersistentVolumeLabel
{% endhighlight %}




