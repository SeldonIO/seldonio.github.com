---
layout: default
title: Kubernetes on Google Cloud
---

# Installing a Kubernetes Cluster on Google Cloud

First of all you need to create a google account, then sign in to Google Cloud Platform Console ([console.cloud.google.com](http://console.cloud.google.com/)).


# Creating a Project

Create a new project and write down the project ID.

Enable billing in the Developers Console in order to use Google Cloud resources.

New users of Google Cloud Platform receive a [$300 free trial](https://console.developers.google.com/billing/freetrial?hl=en). Running through this example shouldn’t cost you more than a few dollars of that trial. Google Container Engine pricing is documented [here](https://cloud.google.com/container-engine/pricing).


# Installing Docker, Google Cloud SDK and Kubectl

Install [Docker](https://docs.docker.com/engine/installation/), and [Google Cloud SDK](https://cloud.google.com/sdk/).
Finally, after Google Cloud SDK installs, run the following command to install kubectl:

{% highlight bash %}
gcloud components install kubectl
{% endhighlight %}

# Creating a cluster

Go to the developers console. On the left hand side menu select Container Engine.

Click on New Container Cluster. Set the name to seldon-server-cluster. Select a cluster size of 1 and a Machine type 4 vCPU with 15GB of memory.

Configure kubectl to use the cluster you just created by running:

{% highlight bash %}
gcloud container clusters get-credentials seldon-server-cluster
{% endhighlight %}


# Installing Seldon on your Cluster

At this point the process is identical to the [regular seldon installation](http://docs.seldon.io/install.html).
kubectl is now linked to your cluster on google cloud so all commands can be used as usual.


# Deleting a cluster

Delete the cluster you created with the following command:

{% highlight bash %}
gcloud container clusters delete seldon-server-cluster
{% endhighlight %}
