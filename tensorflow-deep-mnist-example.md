---
layout: default
title: Tensorflow Deep MNIST Tutorial
---

# Tensorflow Deep MNIST Advanced Tutorial

This example will take you through creating a microservice that recognizes numbers between 0 and 9, based on the CNN model from the [tensorflow deep MNIST demo](https://www.tensorflow.org/versions/r0.10/tutorials/mnist/pros/index.html). 
In this example we will go through every single step for packaging your model into a fully functional seldon microservice operating in the cloud. If you are just interested in testing the prepackaged docker image check out [this tutorial](tensorflow-deep-mnist-example-docker.html).

 * [Create Model](#model)
 * [Save as a pipeline](#pipeline)
 * [Create script to start microservice](#microservice)
 * [Create the docker image](#docker-image)
 * [Pushing the image to your docker hub](#docker-hub)
 * [Creating a seldon kubernetes cluster](#cluster)
 * [Launching the microservice on your cluster](#starting)
 * [Testing your service](#test)


## Create Model<a name="model"></a>

Let's start by creating a model using tensorflow. We are going to implement the digit recognition model that can be found in the [tensorflow advanced tutorial](https://www.tensorflow.org/versions/r0.10/tutorials/mnist/pros/index.html), using the python implementation of tensorflow and seldon.

{% highlight python %}
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
{% endhighlight %}

Here we define a convolutional neural network and train it over the MNIST dataset shipped with tensorflow.


## Save as a pipeline<a name="pipeline"></a>

Now that we have a fully functional machine learning model, we are going to turn it into a pipeline and package it as a microservice using Seldon tools. We can do all of this easily in python:

{% highlight python %}
	from seldon.tensorflow_wrapper import TensorFlowWrapper
	from sklearn.pipeline import Pipeline
	from seldon.pipeline.util import Pipeline_wrapper


	tfw = TensorFlowWrapper(sess,tf_input=x,tf_output=y_conv,tf_constants=[(keep_prob,1.0)],target="y",target_readable="class",excluded=['class'])

	p = Pipeline([('deep_classifier',tfw)])

	pw = Pipeline_wrapper()

	pw.save_pipeline(p,'deep_mnist_pipeline')
{% endhighlight %}

Let's look in more detail at each one of these steps

{% highlight python %}
tfc = TensorFlowWrapper(sess,x,y,target="y",target_readable="class",excluded=['class'])
{% endhighlight %}

Here we use the TensorFlowWrapper class to standardise our model so that it can be integrated in a generic machine learning pipeline (next step). We need to pass as arguments:

 * the tensorflow session (sess)
 * the input variable (x)
 * the output variable (y)
 * a list of the variables that need to take a constant value at prediction time (keep_prob must be set as 1.0)

{% highlight python %}
p = Pipeline([('deep_classifier',tfw)])
{% endhighlight %}

Here we create a scikit learn pipeline comprising a single step which corresponds to our tensorflow model. More information on predictive pipelines can be found [here](prediction-guide.html).

{% highlight python %}
	pw = Pipeline_wrapper()
	pw.save_pipeline(p,'deep_mnist_pipeline')
{% endhighlight %}

Finally we use seldon's pipeline wrapper to save the pipeline to the disk. The microservice will be generated from this save file.

## Create script to start microservice<a name="microservice"></a>

We are going to write a simple bash script to launch the microservice and save it in a file called run_microservice.sh

{% highlight bash %}
#!/bin/bash

set -o nounset
set -o errexit

python /home/seldon/scripts/start_prediction_microservice.py --pipeline /home/seldon/deep_mnist_pipeline --model_name deep_mnist_tensorflow
{% endhighlight %}

This uses seldon's script to launch a microservice from the saved pipeline.
You can try to run it but be aware that it needs for your pipeline to be saved in /home/seldon.

## Creating the docker image<a name="docker-image"></a>

We are going to create a docker image from our microservice. Here is the content of the dockerfile:

{% highlight Dockerfile %}
FROM seldonio/pyseldon:2.1

COPY deep_mnist_pipeline /home/seldon/deep_mnist_pipeline
COPY run_microservice.sh /run_microservice.sh

CMD ["/run_microservice.sh"]
{% endhighlight %}

Let's look at it line by line:

{% highlight Dockerfile %}
FROM seldonio/pyseldon:2.1
{% endhighlight %}

We are going to base our image on pyseldon's image that has seldon installed as well as python and a number of libraries like tensorflow.

{% highlight Dockerfile %}
COPY deep_mnist_pipeline /home/seldon/deep_mnist_pipeline
COPY run_microservice.sh /run_microservice.sh
{% endhighlight %}

We copy the pipeline and the script that starts the microservice

{% highlight Dockerfile %}
CMD ["/run_microservice.sh"]
{% endhighlight %}

And finally, we call our script to launch the microservice!

Now you can choose a name for your image (we went for deep_mnist) and ask docker to build it using the following command line:

{% highlight bash %}
docker build -t <your_userid>/deep_mnist:1.0 .
{% endhighlight %}

seldonio is the name of our docker hub (more about this in the next part) and 1.0 is the version of the image.

## Pushing the image to your docker hub<a name="docker-hub"></a>

In order for your kubernetes cluster on google cloud (or any cloud service) to find your docker image it needs to be pushed to a docker hub. You can create a docker hub very easily [here](https://docs.docker.com/v1.8/userguide/dockerrepos/).
Once you have a docker hub user id you can use the following command to log into it and push your image:

{% highlight bash %}
docker login -u <your_userid> && docker push <your_userid>/deep_mnist:1.0
{% endhighlight %}


## Creating a seldon kubernetes cluster<a name="cluster"></a>

The full documentation for this step can be found [here](install.html)


## Launching the microservice on your cluster<a name="starting"></a>

The first step is to create a client for your microservice. More information on seldon clients can be found [here](seldon-cli.html#client).
This can be done using seldon-cli as follows:

{% highlight bash %}
seldon-cli client --action setup --db-name ClientDB --client-name deep_mnist_client
{% endhighlight %}

This requires an existing datasource. ClientDB is a datasource that is created by seldon on start-up but you can use another one that you create using [seldon-cli db](seldon-cli.html#db).

Finally, you can launch your microservice using kubernetes/bin/start-microservice.

{% highlight bash %}
start-microservice --type prediction --client deep_mnist_client -p tensorflow-deep-mnist /seldon-data/seldon-models/tensorflow_deep_mnist/1/ rest 2.1
{% endhighlight %}


## Testing your service<a name="test"></a>

The microservice takes as input a vector of 784 floats corresponding to the pixels of a 28x28 image and returns a list of probabilities for each number between 0 and 9. In order to test it you can use the [flask webapp](tensorflow-deep-mnist-webapp.html) we have created for this purpose.






