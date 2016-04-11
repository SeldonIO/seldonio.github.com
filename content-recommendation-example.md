---
layout: default
title: Recomendation Example
---

# Creating a Recommendation Service

This example will take you through creating a simple recommendation service for the the [Reuters-21578](http://www.daviddlewis.com/resources/testcollections/reuters21578/) newswire dataset. It will create a simple static document similarity algorithm run as a microservice through the Seldon API server.

 * [Create Reuters client and meta-data](#meta-data)
 * [Create recommendation model](#model)
 * [Create microservice](#microservice)
 * [Serve recommendations](#recommendations)


# Create Reuters client<a name="meta-data"></a>
The first step is to create a Reuters Seldon client and import the meta-data for the items we want to recommend into the Seldon database. These tasks are all packaged into a Docker container ```seldonio/examples-reuters-data``` whose source can be found in ```docker/examples/reuters/data```. The tasks run are:

 * Create a JSON file describing the meta-data attributes
 * Create a csv of the meta data from the downloaded Reuters data
 * Use the seldon-cli to:
    * Create a client "reuters"
    * Upload its meta-data definitions into the db
    * Load the meta-data itself into the db

The Docker container can be run as a on-off Kubernetes job to carry out the tasks:

{% highlight bash %}
cd kubernetes/conf/examples/reuters/
kubectl create -f import-data-job.json
{% endhighlight %}

Check for successful completion using ```kubectl get jobs``` which should show:
{% highlight bash %}
NAME                  DESIRED   SUCCESSFUL   AGE
reuters-import-data   1         1            1m
{% endhighlight %}

You can delete the job once its run with ```kubectl delete -f import-data-job.json```

# Create Recommendaton Model<a name="model"></a>
For this simple example we will build a document similarity model to provide recommendations for newswire article given the current newswire article a user is reading. For this we will build a [gensim](https://radimrehurek.com/gensim/) document similarity model. The detailed steps to build the model can be followed in a [Jupyter notebook](https://github.com/SeldonIO/seldon-server/blob/master/python/examples/doc_similarity_reuters.ipynb). These have been packaged into a docker container ```seldonio/reuters-example```.

In general, Seldon provides several content recommendation models in Spark as well as python based modules.

# Create Microservice<a name="microservice"></a>
To serve predictions from our model we need to run the repackaged container which exposes a miroservice endpoint for the model scoring. To start any microservice and connect it to a client using the script ```kubernetes/bin/run_recommendation_microservice.sh``` which takes 4 arguments:

  * A name for the microservice
  * An image to pull that can be run to start the microservice
  * A version for the image
  * A client to connect the microservice to

The script create a Kubernetes deployment for the microservice in ```kubernetes/conf/microservices```. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.

To start the Reuters gensim model serving run:

{% highlight bash %}
run_recommendation_microservice.sh reuters-example seldonio/reuters-example 1.0 reuters
{% endhighlight %}

The script will create the Kubernetes Deployment and use the seldon-cli to update the "reuters" client to add the microservice as a runtime algorithm. 

# Serve Recommendations<a name="recommendations"></a>
You can now call the Seldon server using the Seldon CLI to test recommendations:
{% highlight bash %}
seldon-cli --quiet api --client-name reuters --endpoint  /js/recommendations --item 6020 --limit 5
{% endhighlight %}

The response should be like:

{% highlight json %}
{"size":5,"requested":5,"list":[{"id":"5762","name":"","type":1,"first_action":1460104099000,"last_action":1460104099000,"popular":false,"demographics":[],"attributes":{},"attributesName":{"recommendationUuid":"21","title":"AERO SERVICES <AEROE> IN PACT FOR NOMINATIONS","body":"Aero Services International Inc\nsaid it signed an agreement with Dibo Attar, who controls about\n39 pct of its common stock, under which three nominees to\nAero's board have been selected by Attar.\n    In addition to Attar, the nominees are Stephen L. Peistner,\nchairman and chief executive officer of <McCrory Corp> and\nJames N.C. Moffat III, vice president and secretary of\n<Eastover Corp>.\n Reuter\n\u0003"}},{"id":"8571","name":"","type":1,"first_action":1460104099000,"last_action":1460104099000,"popular":false,"demographics":[],"attributes":{},"attributesName":{"recommendationUuid":"21","title":"AVALON <AVL> STAKE SOLD BY DELTEC","body":"Avalon Corp said that <Deltec\nPanamerica SA> has arranged to sell its 23 pct stake in Avalon\nand that Deltec's three representatives on Avalon's board had\nresigned.\n    An Avalon spokeswoman declined to indentify the buyer of\nDeltec's stake or give terms of the sale.\n    In addition, Avalon said three other directors resigned. It\nsaid Benjamin W. Macdonald, a director of <TMOC Resources Ltd>,\nthe principal holder of Avalon stock, and Hardwick Simmons, a\nvice chairman of Shearson Lehman Bros Inc, were then named to\nthe board.\n Reuter\n\u0003"}},{"id":"7816","name":"","type":1,"first_action":1460104099000,"last_action":1460104099000,"popular":false,"demographics":[],"attributes":{},"attributesName":{"recommendationUuid":"21","title":"CELINA <CELNA> SHAREHOLDERS APPROVE SALE","body":"Celina Financial Corp said\nshareholders at a special meeting approved a transaction in\nwhich the company transferred its interest in three insurance\ncompanies to a wholly owned subsidiary which then sold the\nthree companies to an affiliated subsidiary.\n    It said the company's interests in West Virginia Fire and\nCasualty Co, Congregation Insurance co and National Term Life\nInsurance Co had been transferred to First National Indemnity\nCo, which sold the three to Celina Mutual for cash, an office\nbuilding and related real estate.\n Reuter\n\u0003"}},{"id":"18963","name":"","type":1,"first_action":1460104099000,"last_action":1460104099000,"popular":false,"demographics":[],"attributes":{},"attributesName":{"recommendationUuid":"21","title":"ALLEGHENY <AI> SELLS THREE INDUSTRIAL UNITS","body":"Allegheny International Inc said it\nsold three of its industrial units which served the railroad\nindustry to <Chemetron Railway Products Inc>, a senior\nmanagement group of Allegheny.\n    Terms of the transaction were not disclosed.\n    Included in the sale were Chemetron Railway Products, True\nTemper Railway Appliances Inc and Allegheny Axle Co, the\ncompany said.\n    The three units include 12 plants throughout the U.S., the\ncompany said.\n Reuter\n\u0003"}},{"id":"9431","name":"","type":1,"first_action":1460104099000,"last_action":1460104099000,"popular":false,"demographics":[],"attributes":{},"attributesName":{"recommendationUuid":"21","title":"SYNALLOY <SYO> ENDS PLANS TO SELL UNIT","body":"Synalloy Corp said it has\nended talks on the sale of its Blackman Uhler Chemical Division\nto Intex Products Inc because agreement could not be reached.\n    The company said it does not intend to seek another buyer.\n Reuter\n\u0003"}}]}
{% endhighlight %}