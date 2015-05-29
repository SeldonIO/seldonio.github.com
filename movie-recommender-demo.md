---
layout: default
title: Movie Recommender Demo
---

# Movie Recommender Demo
Seldon provides a demonstration of a movie recommender using the [Movielens 10 Million dataset](http://grouplens.org/datasets/movielens/) plus some extra data from [Hetrec 2011 dataset](http://grouplens.org/datasets/hetrec-2011/) and some data sourced from Freebase by Seldon. These data sources are used to train four different models that can be used to provide movie recommendations based on a viewing history of a user. The demo can be viewed [online](http://www.seldon.io/movie-demo/) and comes with the Seldon VM. These docs will describe the steps to create the demo from inside the VM.

The dataset is of reasonable size so you will need at least 4G of RAM for creating all the recommendation models.


## Quick Start

1. Install and ssh into the Seldon [VM](vm.html) or [AWS AMI](vm-aws.html).

1. Determine and setup the Embedly key usage if necessary. Otherwise the images will not show correctly.
If you plan to view the demo via a url other than 'localhost' then you have two choices (Note you will be fine on the Vagrant VM so can skip this step):

   **option 1**: Use an Embedly key.
   Sign up at "http://embed.ly/", obtain the key and save in the demo:

     ```
     cd ~/movie-demo-setup
     ```

     ```
     echo "export EMBEDLY_KEY=<your-key>" > run_settings
     ```

   **option 2**: Use a ssh tunnel to obtain "localhost" url. This will be explained later.

1. Create the movie demo

   ```
   cd ~/movie-demo-setup
   ```

   ```
   ./create-movie-demo
   ```

1. Start Apache Tomcat 

   ```
   ~/apps/tomcat/bin/startup.sh
   ```

1. View the demo in your browser:

   ```
   http://localhost:8080/movie-demo/
   ```

If running via the Seldon AMI, using a url other than "localhost" will result in no images being shown if no Embedly key was used when creating the demo.

To display the images in case use a ssh tunnel to obtain "localhost" url. Use the following in a terminal to forward port 8080 from your remote host to your localhost:
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


