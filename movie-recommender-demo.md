---
layout: default
title: Movie Recommender Demo
---

# Movie Recommender Demo
Seldon provides a demonstration of a movie recommender using the [Movielens 10 Million dataset](http://grouplens.org/datasets/movielens/) plus some extra data from [Hetrec 2011 dataset](http://grouplens.org/datasets/hetrec-2011/) and some data sourced from Freebase by Seldon. These data sources are used to train four different models that can be used to provide movie recommendations based on a viewing history of a user. The demo can be viewed [online](http://www.seldon.io/movie-demo/). These docs will describe the steps to create the demo fram scratch.

If you are running the Seldon VM, it comes with the demo already pre built. If you run the scripts below it will recreate everything from scratch.

The dataset is of reasonable size so there are certain time and system requirement caveats:

 * At least 5GB of RAM

Most of the model creation is fast except for the item_similarity model. On a ThinkPad T440P the various models take roughly:

 * matrix_factorization : 10 mins
 * tag similarity : 2 mins
 * item_similarity : 1.5 hours
 * word2vec : 10 mins

To decrease the time taken you can comment out the item_similarity model from the create_recommender_demo.sh, see the "In Depth Section" below.

## Quick Start

 1. Download and Start the Seldon VM as describe [here](vm.html)
 1. Run:
  {% highlight bash %}
cd dist/movie_recommender_demo
./create_recommender_demo.sh 
  {% endhighlight %}

This will:

 1. Download all the required data
 1. Combine the data and store it in a Seldon database (for movie meta data and existing users) and historical actions file (for the ratings by users of movies)
 1. Create the various models
 1. Deploy them to the running Seldon API server

The rebuilt movie demo will again be available at:

        http://127.0.0.1:8080/movie-demo/


## In Depth Steps
