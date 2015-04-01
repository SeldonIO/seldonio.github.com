---
layout: default
title: External Algorithms
---

# External Algorithms

(comming soon)

## External Python Recommender

The Seldon Server can utilize algorithms written in python and served up by the External Python Recommender.

To use this recommender, follow these steps:

1. Install python dependencies.

        pip install Flask
        pip install pylibmc
        pip install gunicorn

1. Clone the Seldon Server project.

        git clone https://github.com/SeldonIO/seldon-server.git

1. Navigate to the python recommender scripts.

        cd seldon-server/external-recommender/python

1. Copy or edit the "**example_alg.py**" script, and customize the "**get_recommendations**" function.

    The paramters to the "get_recommendations" function is as follows:

    * **user_id** : a long for the user id
    * **item_id** : a long for the item id
    * **client** : a str for the client
    * **recent_interactions_list** : a list of item ids
    * **data_set** : a set of item ids
    * **limit** : an int for the number of recommendations to return

1. Modify the script "**recommender_config.py**".  
    Example:

        RECOMMENDER_ALG="example_alg"
        MEMCACHE={
            "servers": ["192.168.59.103:11211"],
            "pool_size" : 1
        }

    Change **RECOMMENDER_ALG** to the name of the script with the custom "get_recommendations" function, without the .py extension.

    Change **MEMCACHE** to your memcache settings.

1. Serve the recommender for testing.

        python recommender.py

1. Serve the recommender using gunicorn for handling concurrent requests.  
    Example

        gunicorn -w 4 -b 127.0.0.1:5000 recommender:app

