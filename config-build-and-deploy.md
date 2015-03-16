---
layout: default
title: Build and deploy algorithm configuration
---

Algorithm configuration is added via ZooKeeper as per the [configuration page]().

# Default configuration

Provided below is a default set of algorithm configurations to add. Put this at `/config/default_strategy` 

{% highlight javascript %}
{"algorithms":[{"name":"mfRecommender","tag":"MATRIX_FACTOR","includers":["mostPopularIncluder"],"filters":[],"config":{"items_per_includer":200}},{"name":"recentItemsRecommender","tag":"RECENT_ITEMS","includers":[]}],"combiner":"firstSuccessfulCombiner"}
{% endhighlight %}

This provides a base set of algorithms. If you want to alter this then please refer to the configuration page.


