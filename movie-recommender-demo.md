---
layout: default
title: Movie Recommender Demo
---

# Movie Recommender Demo
Seldon provides a demonstration of a movie recommender using the [Movielens 10 Million dataset](http://grouplens.org/datasets/movielens/) plus some extra data from [Hetrec 2011 dataset](http://grouplens.org/datasets/hetrec-2011/) and some data sourced from Freebase by Seldon. These data sources are used to train four different models that can be used to provide movie recommendations based on a viewing history of a user. The demo can be viewed [online](http://www.seldon.io/movie-demo/) and comes prebuilt with the Seldon VM. These docs will describe the steps to create the demo from scratch.

The dataset is of reasonable size so there are certain time and system requirement caveats:

 * At least 5G of RAM for recreating all models and running the Seldon containers inside a VM

Most of the model creation is fast except for the item_similarity model. On a ThinkPad T440P with a good network connection the download and model creation take roughly:

 * download and data ETL 10 mins
 * matrix_factorization : 5 mins (2G RAM)
 * tag similarity : 2 mins (<1G RAM)
 * item_similarity : 10 mins (running on a random 25% of the full set of actions) (2G RAM)
 * word2vec : 5 mins (2G RAM)


## Quick Start

 1. Download and start the Seldon VM as described [here](vm.html)
 1. Run:
  {% highlight bash %}
cd dist/movie_recommender_demo
./create_recommender_demo.sh 
  {% endhighlight %}

This will:

 1. Download all the required data
 1. Combine the data and store it in a Seldon database (for movie meta data and existing users) and create a historical actions file (for the ratings by users of movies)
 1. Create the various models
 1. Deploy them to the running Seldon API server

The rebuilt movie demo will again be available at:

        http://127.0.0.1:8080/movie-demo/


## In Depth Steps
The steps to create the content recommendation models for the Movielens based data are described below. These steps will hopefully help you gain an understanding of how to injest and model a reasonable large and complex dataset.

### Data Download and ETL
Three data sources are used to create the demo

 1. [Movielens 10 Million dataset](http://grouplens.org/datasets/movielens/)
 1. [Hetrec 2011 dataset](http://grouplens.org/datasets/hetrec-2011/)
 1. Some data sourced from Freebase by Seldon by matching movie names and year release to freebase entries.

The first step is to download the two external datasets and then merge the data to produce csv and JSON files in the correct format for the generic Seldon item and user import scripts, see [here](data.html) for further details. We take the core data from the Movielens dataset and add directors and actors matched via Freebase queries and finally add the IMDB image for movies from the HetRec dataset. The resulting JSON file describing the item attributes is shown below. For top_tags we take the top 3 tags from the Movielens dataset and add actor and directors from freebase. From a small amount of testing this reduced tag set seemed to give best results for the tag similarity model.

  {% highlight json %}
{"types":[
   {"type_attrs": [
      {"name": "title", "value_type": "string"},
	  {"name": "img_url", "value_type": "string"},
	  {"name": "top_tags", "value_type": "text"},
	  {"name": "movielens_tags_full", "value_type": "text"},
	  {"name": "actors", "value_type": "string"},
	  {"name": "directors", "value_type": "string"}
	  ],
   "type_id": 1,
   "type_name": "movie"}]}
  {% endhighlight %}

The csv file for items contains all the above attributes plus "id" and "name". These two attributes are special in that they don't need to be defined in the json and will be used to populate the items table in the Seldon db. The first line from the items csv file created is shown below:

  {% highlight bash %}
id,name,title,img_url,top_tags,movielens_tags_full,actors,directors
1,Toy Story (1995),Toy Story (1995),"http://ia.media-imdb.com/images/M/MV5BMTMwNDU0NTY2Nl5BMl5BanBnXkFtZTcwOTUxOTM5Mw@@._V1._SX214_CR0,0,214,314_.jpg","pixar,animation,animation,tom_hanks,don_rickles,jim_varney,john_lasseter","pixar,animation,disney,toys,computer_animation,cgi,comedy,tim_allen,time_travel,animated,cartoon,adventure,children,toy,tom_hanks,classic,family,avi,disney_animated_feature,want_to_see_again,imdb_top_250,the_boys,toys_come_to_life,kids_movie,unlikely_friendships,action_figures,buzz_lightyear,light,action_figure,first_cgi_film,lots_of_heart,almost_favorite,tumey's_to_see_again,warm,national_film_registry,villian_hurts_toys,cg_animation,pixar_animation,clever,fun,rated-g,daring_rescues,witty,3d,rousing,erlend's_dvds,usa,buy,john_lasseter,funny,heroic_mission,ya_boy,woody,want,very_good,fanciful,bright,fantasy,engaging,tumey's_vhs,humorous,buddy_movie,","tom_hanks,don_rickles,jim_varney",john_lasseter
  {% endhighlight %}

There are no user attributes so the user csv file just has user ids taken from the Movielens dataset.

We can now run the generic scripts provided in the seldon-tools container to populate the database as well as create a historical actions JSON file from the Movielens ratings we can use in training the  user activity based models.

### Matrix Factorization Model
The model uses the actions (ratings history) of the users. We use the following settings which we place within Consul:

{% highlight json %}
  {"rank":30, "lambda":0.1, "alpha":1, "iterations":5}
{% endhighlight %}

The resulting model will be found in /seldon-models/movielens/matrix_factorization/1

### Item Similarity Model
Like matrix factorization this model also only uses the actions (ratings history) of the users. The following setting were used:

{% highlight json %}
{"sample:0.25", "limit":100, "dimsum_threshold":0.5}
{% endhighlight %}

Running over the full set of 10 million actions takes some time and around 8G of memory so for a faster build we subsample the actions (0.25) to get roughly 2.5 million random actions (user ratings). If you have cpu power and memory you can remove this to run on the full dataset.

### Tag similarity Model
Tag similarity does not use the user actions but simply the meta-data of the items to create vectors whose cosine similarity can be used to find similar movies. The settings used below, show we train on the top_tags field of the movies which contains the top 3 tags from the Movielens dataset plus main actors and directors taken from Freebase. It would be possible to test on other attributes or combination of attributes.

{% highlight json %}
{"attr_names":"top_tags"}
{% endhighlight %}

The model is fast to create and requires very little RAM.

### Word2vec Model
This model runs on the session histories of all the users movie ratings and runs the word2vec language model on it to create movie vectors that can be compared for similarity. The settings are:

{% highlight json %}
{ "min_word_count":50, "vector_size":30 }
{% endhighlight %}

We ignore movies that have been viewed less than 50 times and create vectors of size 30. 


