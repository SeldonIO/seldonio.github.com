---
layout: default
title: Advanced Recommender Configuration
---

# Advanced Recommender Configuration
The basic configuration for providing recommendation is discused [here](http://docs.seldon.io/runtime-recommendation.html). However, Seldon allows much more complex scenarios to be configured which are discussed in the following sections:

 * [Cascading algorithms](#cascading-algorithms) : try and/or combine several recommendation algorithms in 1 API call
 * [Multi-variant testing](#multi-variant-tests) : start and stop A/B tests or multi-variant tests
 * [API controlled recommendation variants](#recommendation-variants) : use tags sent via the API call to choose from a selection of recommendation strategies
 
## Cascading Algorithms<a name="cascading-algorithms"></a>
Not all recommender algorithms may be applicable to all users. For example a matrix factorization model may only be created for users with previous actions. Therefore its good practise to have a set of algorithms that can be cascaded through until one succeeds for the current user. This can be easily handled within Seldon by provided the ordered list of algorithms in the configuration, for example:

{% highlight json %}
{
  "algorithms":
  [
   {"name":"mfRecommender","filters":[],"includers":[],"config":[]},
   {"name":"globalClusterCountsRecommender","filters":[],"includers":[],"config":[]}
  ],
  "combiner":"firstSuccessfulCombiner"
  }
{% endhighlight %}

In the above example, we will try the mfRecommender and if that fails to return recommendations we will try a recommender that returns global most popular items from the cluster algorithm.

Another scenario is that you may wish to combine the results of several recommenders. This can be accomplished by changing the ***combiner*** to one of the following choices:

 * rankSumCombiner : combine multiple recommendations using the rank of each item returned in each recommenders list of recommendations. The number of algorithms combined defaults to two but can be configured with the config option combiner.maxResultSets
 * scoreOrderCombiner : combine multipe recommendations using the scores returned from each item in each individual recommender.

## Multi Variant Tests<a name="multi-variant-tests"></a>
It is common practice when testing algorithms in live environments to run A/B tests to evaluate the success of different strategies. A test can be defined in zookeeper by setting the path 

{% highlight bash %}
/all_clients/<clientname>/alg_test
{% endhighlight %}

For example, for a client "acme":

{% highlight bash %}
/all_clients/acme/alg_test
{% endhighlight %}

The configuration should provide the algorithm settings for each variant. This includes:
  
 * **config** : the algorithm configuration
 * **label** : a label for this variant that can be referenced in the logs
 * **ratio** : the percentage of traffic that should be sent to this test variant, in range 0-1.

A template for an A/B test is shown below, with the algorithm sections missing:

{% highlight json %}
{
    "variations": [
        {
            "config": {
            },
            "label": "A",
            "ratio": 0.5
        },
        {
            "config": {
            },
            "label": "B",
            "ratio": 0.5
        }
    ]
}
{% endhighlight %}

To switch on the test you set the zookeeper path ```/all_clients/<client_name>/alg_test_switch``` to contain the value ```true```. Similarly to stop a test you set this zookeeper node to contain ```false```.

Multi-variant tests can be set up in the same manner by simply extending the number of variations and changing the ratios as appropriate. The results of impressions and clicks will appear in the ctr-alg.log in the Seldon log folder. Seldon's codebase contains Spark algorithms to process these logs once they have been pushed to a central location by fluentd and produce CTR analytics.

## API Controlled Recommendion Variants and Tests<a name="recommendation-variants"></a>
There are many situations where you want to control the algorithms run via the API itself, for example:
  
 * To have multiple recommendations per page, e.g. in-section recommendations and site-wide recommendations
 * To serve different recommendations for mobile users as opposed to desktop users
 * To run a multi-variant test on an API determined set of users, e.g. users who view a certain subsection of a web-site

These cases can be handled by passing a recommendation tag in the API call. The added parameter is ```recTag``` and it should contain a string keyword. This tag is matched against the configuration for a client and the apporpriate set of algorithms is run. If no match is found then a default set of algorihtms is run. An example configuration for this is shown in outline below:

{% highlight json %}

{
    "defaultStrategy": {
    },
    "recTagToStrategy": {
        "insection": {	
	}
        "mobile": {
	}
        "sitewide": {
        }
    }
}
{% endhighlight %}

In this example there is a default strategy in case a rec tag is not found and 3 other strategies, one for "mobile", one for "sitewide" and one for "insection". The config (not shown) within each of these can be an algorithms configuration or a test variant. This means quite complex configurations can be created. For example, a baseline and normal test running for in-section and sitewide recommendations but a fixed set of algorithms always for mobile. To expand this example out some more this would look like:

{% highlight json %}
{
    "defaultStrategy": {
    },
    "recTagToStrategy": {
        "category": {
            "variations": [
                {
                    "config": {
                    },
                    "label": "baseline",
                    "ratio": 0.05
                },
                {
                    "config": {
                    },
                    "label": "normal",
                    "ratio": 0.95
                }
            ]
        },
        "mobile": {
	 },
        "sitewide": {
            "variations": [
                {
                    "config": {
                    },
                    "label": "baseline",
                    "ratio": 0.05
                },
                {
                    "config": {
                    },
                    "label": "normal",
                    "ratio": 0.95
                }
            ]
        }
    }
}
{% endhighlight %}

In the above example we have baseline algorithms running for 5% of the traffic for sitewide and insection recommendations while 95% of the traffic for these recommandation tags gets the "normal" recommendations. For mobile there is a fixed set of algorithms. 




