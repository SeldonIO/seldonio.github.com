---
layout: default
title: Movie Recommender Demo
---

# Movie Recommender Demo
Seldon provides a demonstration of a movie recommender using the [Movielens 10 Million dataset](http://grouplens.org/datasets/movielens/) plus some extra data from [Hetrec 2011 dataset](http://grouplens.org/datasets/hetrec-2011/) and some data sourced from Freebase by Seldon. These data sources are used to train four different models that can be used to provide movie recommendations based on a viewing history of a user. The demo can be viewed [online](http://www.seldon.io/movie-demo/) and comes prebuilt with the Seldon VM. These docs will describe the steps to create the demo from scratch.

The dataset is of reasonable size so you will need at least 4G of RAM for creating all the recommendation models.


## Quick Start

 1. Install and ssh into the Seldon [VM](vm.html) or [AWS AMI](vm-aws.html) and then:
{% highlight bash %}
cd ~/movie-demo-setup
./create-movie-demo
{% endhighlight %}

 1. Start Apache Tomcat 
{% highlight bash %}
cd ~/apps/apache-tomcat-7.0.61/bin
./startup.sh
{% endhighlight %}        

 1. View the demo in your browser:
{% highlight http %}
        http://localhost:8080/movie-demo/
{% endhighlight %}        


**IMPORTANT**: If running via the Seldon AMI, using a url other than "localhost" will result in no images being shown.  
To display the images there are two options:

**option 1**: Use an Embedly key.
Sign up at "http://embed.ly/", obtain the key and re-start using the following:
{% highlight bash %}
cd ~/seldon/dist
export EMBEDLY_KEY=<your-key>
./start-all
{% endhighlight %}        

**option 2**: Use a ssh tunnel to obtain "localhost" url. Use the following in a terminal to formard port 8080 from your remote host to your localhost:
{% highlight bash %}
ssh -i <path-to-your-pem-file> -L 8080:localhost:8080 ubuntu@<your-remote-host>
{% endhighlight %}        
Leave that open and use the following in a browser:
{% highlight http %}
http://localhost:8080/movie-demo/
{% endhighlight %}        
An example screenshot is shown below:

![Movie Demo](/img/movie-demo.png)

## Seldon API Explorer
The Seldon REST and Javascript APIs can be explored using:
{% highlight http %}
        http://localhost:8080/swagger/
{% endhighlight %}        


