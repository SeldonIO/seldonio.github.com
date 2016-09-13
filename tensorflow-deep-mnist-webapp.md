---
layout: default
title: Tensorflow Deep MNIST Webapp
---

# Deep MNIST Webapp

This documentation will show you how to setup the Deep MNIST webapp on your Kubernetes cluster. The webapp is a flask application with a javascript front-end. It takes as input a drawing from the user and outputs predictions that it gets from the [seldon deep MNIST microservice](tensorflow-deep-mnist-example-docker.html).

## Prerequisites

 * You have [installed Seldon](install.html) on a Kubernetes cluster
 * You have setup the [Tensorflow Deep MNIST microservice](tensorflow-deep-mnist-example-docker.html) on your cluster, on a client called "deep-mnist-client".

## Getting the client key and secret

The webapp will communicate with the client via a REST API. It needs to authenticate using a client key and secret. We will use the seldon cli to obtain the key and secret.

```bash
seldon-cli keys  --client-name deep-mnist-client
```

More detail on how to use the seldon cli can be found in our [seldon-cli documentation](seldon-cli.html).

## Getting the IP address of seldon-server

To find out the internal IP address of the seldon-server container in your cluster use the following command:

```bash
kubectl get services seldon-server
```

You will need to give this IP address to your webapp in the next step so that it knows where to query for predictions.

## Start the webapp

The webapp is available prepackaged in a docker container ```seldonio/deep_mnist_webapp``` on dockerhub. The source code can be found [here](https://github.com/SeldonIO/deep-mnist-webapp). We are going to start the webapp from  the docker image using the following command:

```bash
kubectl run deep-mnist-webapp --image=seldonio/deep-mnist-webapp:latest --port=80 "/run_webapp.sh <seldon-server-ip> <key> <secret>"
```

Now we need to expose port 80 so that the webapp can be accessed outside your cluster:

```bash
kubectl expose deployment/deep-mnist-webapp --type="LoadBalancer"
```

## Try it out!

This is it, you should now be able to access the webapp on your browser. First, find out the external IP address of the webapp by running:

```bash
kubectl get services deep-mnist-webapp
```

Now you can just go to ```http://<external-ip>:80``` in your favorite browser.

