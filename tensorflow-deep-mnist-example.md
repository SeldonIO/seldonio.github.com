---
layout: default
title: Tensorflow Deep MNIST Tutorial
---

# Tensorflow Deep MNIST Tutorial

Contents:
 * Create Model
 * Save as a pipeline
 * Create script to start microservice
 * Create the docker image
 * Pushing the image to your docker hub
 * Creating a seldon kubernetes cluster
 * Launching the microservice on your cluster


## Create Model

Let's start by creating a model using tensorflow. We are going to impoement the digit recognition model that can be found in [tensorflow advanced tutorial](https://www.tensorflow.org/versions/r0.10/tutorials/mnist/pros/index.html).

```python

from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot = True)
import tensorflow as tf

def weight_variable(shape):
  initial = tf.truncated_normal(shape, stddev=0.1)
  return tf.Variable(initial)

def bias_variable(shape):
  initial = tf.constant(0.1, shape=shape)
  return tf.Variable(initial)

def conv2d(x, W):
  return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
  return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
                        strides=[1, 2, 2, 1], padding='SAME')

if __name__ == '__main__':
	
	x = tf.placeholder(tf.float32, [None,784])

	W_conv1 = weight_variable([5, 5, 1, 32])
	b_conv1 = bias_variable([32])

	x_image = tf.reshape(x, [-1,28,28,1])

	h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
	h_pool1 = max_pool_2x2(h_conv1)

	W_conv2 = weight_variable([5, 5, 32, 64])
	b_conv2 = bias_variable([64])

	h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
	h_pool2 = max_pool_2x2(h_conv2)

	W_fc1 = weight_variable([7 * 7 * 64, 1024])
	b_fc1 = bias_variable([1024])

	h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
	h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)

	keep_prob = tf.placeholder(tf.float32)
	h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)

	W_fc2 = weight_variable([1024, 10])
	b_fc2 = bias_variable([10])

	y_conv=tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)

	y_ = tf.placeholder(tf.float32, [None, 10])


	cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y_conv), reduction_indices=[1]))

	train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)

	correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
	accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

	init = tf.initialize_all_variables()

	sess = tf.Session()
	sess.run(init)

	for i in range(20000):
	    batch_xs, batch_ys = mnist.train.next_batch(50)
	    if i%100 == 0:

    		train_accuracy = accuracy.eval(session=sess,feed_dict={x:batch_xs, y_: batch_ys, keep_prob: 1.0})
    		print("step %d, training accuracy %.3f"%(i, train_accuracy))
	    sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys, keep_prob: 0.5})

	print("test accuracy %g"%accuracy.eval(session=sess,feed_dict={x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))


```

Here we define a convolutional neural netwrok and train it over the MNIST dataset shipped with tensorflow.


## Save as a pipeline

Now that we have a fully functional machine learning model, we are going to turn it into a pipeline and package it as a microservice using Seldon tools. We can do all of this easily in python:

```python

	from seldon.tensorflow_wrapper import TensorFlowWrapper
	from sklearn.pipeline import Pipeline
	from seldon.pipeline.util import Pipeline_wrapper


	tfw = TensorFlowWrapper(sess,tf_input=x,tf_output=y_conv,tf_constants=[(keep_prob,1.0)],target="y",target_readable="class",excluded=['class'])

	p = Pipeline([('deep_classifier',tfw)])

	pw = Pipeline_wrapper()

	pw.save_pipeline(p,'deep_mnist_pipeline')

```

Let's look in more detail at each one of these steps

```python

tfc = TensorFlowWrapper(sess,x,y,target="y",target_readable="class",excluded=['class'])

```

Here we use the TensorFlowWrapper class to standardise our model so that it can be integrated in a generic machine learning pipeline (next step). We need to pass as arguments:
 * the tensorflow session (sess)
 * the input variable (x)
 * the output variable (y)
 * a list of the variables that need to take a constant value at prediction time (keep_prob must be set as 1.0)

```python

p = Pipeline([('deep_classifier',tfw)])

```

Here we create a scikit learn pipeline comprising a single step which corresponds to our tensorflow model. More information on predictive pipelines can be found [here](prediction-guide.html).

```python

	pw = Pipeline_wrapper()
	pw.save_pipeline(p,'deep_mnist_pipeline')

```

Finally we use seldon's pipeline wrapper to save the pipeline to the disk. The microservice will be generated from this save file.

## Create script to start microservice

We are going to write a simple bash script to launch the microservice and save it in a file called run_microservice.sh

```bash

#!/bin/bash

set -o nounset
set -o errexit

python /home/seldon/scripts/start_prediction_microservice.py --pipeline /home/seldon/deep_mnist_pipeline --model_name deep_mnist_tensorflow

```

This uses seldon's script to launch a microservice from the saved pipeline.
You can try to run it but be aware that it needs for your pipeline to be saved in /home/seldon.

## Creating the docker image

We are going to create a docker image from our microservice. Here is the content of the dockerfile:

```Dockerfile

FROM seldonio/pyseldon:2.0.5

COPY deep_mnist_pipeline /home/seldon/deep_mnist_pipeline
COPY run_microservice.sh /run_microservice.sh

RUN conda install -y -c conda-forge tensorflow

CMD ["/run_microservice.sh"]

```

Let's look at it line by line:

```Dockerfile

FROM seldonio/pyseldon:2.0.5

```

We are going to base our image on pyseldon's image that has seldon installed as well as python and a number of libraries.

```Dockerfile

COPY deep_mnist_pipeline /home/seldon/deep_mnist_pipeline
COPY run_microservice.sh /run_microservice.sh

```

We copy the pipeline and the script that starts the microservice

```Dockerfile

RUN conda install -y -c conda-forge tensorflow

```

We install tensorflow.

```Dockerfile

CMD ["/run_microservice.sh"]

```

And finally, we call our script to launch the microservice!

Now you can choose a name for your image (we went for deep_mnist) and ask docker to build it using the following command line:

```bash
docker build -t seldonio/deep_mnist:1.0 .
``` 

seldonio is the name of our docker hub (more about this in the next part) and 1.0 is the version of the image.

## Pushing the image to your docker hub

In order for your kubernetes cluster on google cloud (or any cloud service) to find your docker image it needs to be pushed to a docker hub. You can create a docker hub very easily [here](https://docs.docker.com/v1.8/userguide/dockerrepos/).
Once you have a docker hub you can use the following command to log into it and push your image:

```bash
docker login -u seldonio && docker push seldonio/deep_mnist:1.0
```


## Creating a seldon kubernetes cluster

The full documentation for this step can be found [here](install.html)


## Launching the microservice on your cluster

The first step is to create a client for your microservice. More information on seldon clients can be found [here](seldon-cli.html#client).
This can be done using seldon-cli as follows:

```bash

seldon-cli client --action setup --db-name ClientDB --client-name deep_mnist_client

```

This requires an existing datasource. ClientDB is a datasource that is created by seldon on start-up but you can use another one that you create using [seldon-cli db](seldon-cli.html#db).

Finally, you can launch your microservice using kubernetes/bin/run_prediction_microservice.sh.

```bash
run_prediction_microservice.sh deep_mnist_service seldonio/deep_mnist 1.0 deep_mnist_client
```










