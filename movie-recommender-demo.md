---
layout: default
title: Movie Recommender Demo
---

# Movie Recommender Demo
Seldon provides a demonstration of a movie recommender using the [Movielens 10 Million dataset](http://grouplens.org/datasets/movielens/) plus some extra data from [Hetrec 2011 dataset](http://grouplens.org/datasets/hetrec-2011/) and some data sourced from Freebase by Seldon. These data sources are used to train five different models that can be used to provide movie recommendations based on a viewing history of a user. The demo can be viewed online. These docs will describe the steps to create the demo fram scratch.

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

Note: The model creation process can take several hours if all models are created. See the the next section if you wish to edit which models get created.

## In Depth Steps
