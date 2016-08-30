---
layout: default
title: Recomendation Example
---

# Reuters Newswire Recommendation

This example will take you through creating a simple recommendation service for the the [Reuters-21578](http://www.daviddlewis.com/resources/testcollections/reuters21578/) newswire dataset. It will create a simple static document similarity algorithm run as a microservice through the Seldon API server.

 * [Prerequisites](#prerequisites)
 * [Create Reuters client and meta-data](#meta-data)
 * [Create recommendation model](#model)
 * [Create microservice](#microservice)
 * [Serve recommendations](#recommendations)
 * [Next Steps](#next-steps)

# Prerequisites<a name="prerequisites"></a>

 * [You have installed Seldon on a Kubernetes cluster](install.html)
 * You haved added ```seldon-server/kubernetes/bin``` to you shell PATH environment variable.


# Create Reuters client<a name="meta-data"></a>
The first step is to create a Reuters Seldon client and import the meta-data for the items we want to recommend into the Seldon database. These tasks are all packaged into a Docker container ```seldonio/examples-reuters-data``` whose source can be found in ```docker/examples/reuters/data```. The tasks run are:

 * Create a JSON file describing the meta-data attributes
 * Create a CSV of the meta data from the downloaded Reuters data
 * Use the seldon-cli to:
    * Create a client "reuters"
    * Upload its meta-data definitions into the db
    * Load the meta-data itself into the db

The Docker container can be run as a one-off Kubernetes job to carry out the tasks:

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

# Create Recommendation Model<a name="model"></a>
For this simple example we will build a document similarity model to provide recommendations for newswire article given the current newswire article a user is reading. For this we will build a [gensim](https://radimrehurek.com/gensim/) document similarity model. The detailed steps to build the model can be followed in a [Jupyter notebook](https://github.com/SeldonIO/seldon-server/blob/master/python/examples/doc_similarity_reuters.ipynb). These have been packaged into a docker container ```seldonio/reuters-example```.

In general, Seldon provides several content recommendation models in Spark as well as python based modules.

# Create Microservice<a name="microservice"></a>
To serve predictions from our model we need to run the prepackaged docker image which exposes a miroservice endpoint for the model scoring. To start any microservice and connect it to a client using the script ```kubernetes/bin/run_recommendation_microservice.sh``` which takes 4 arguments:

  * A name for the microservice
  * An image to pull that can be run to start the microservice
  * A version for the image
  * A client to connect the microservice to

The script create a Kubernetes deployment for the microservice in ```kubernetes/conf/microservices```. If the microserice is already running Kubernetes will roll-down the previous version and roll-up the new version.

To start the Reuters gensim model serving run:

{% highlight bash %}
run_recommendation_microservice.sh reuters-example seldonio/reuters-example 1.0 reuters
{% endhighlight %}

The script will create the Kubernetes Deployment and use the seldon-cli to update the "reuters" client to add the microservice as a runtime algorithm. Check with ```kubectl get pods -l name=reuters-example``` that the pod running the mircroservice is running.  

# Serve Recommendations<a name="recommendations"></a>
You can now call the Seldon server using the Seldon CLI to test recommendations:
{% highlight bash %}
seldon-cli --quiet api --client-name reuters --endpoint  /js/recommendations --item 6020 --limit 3
{% endhighlight %}

The response should be like:

{% highlight json %}
{
  "size": 3,
  "requested": 3,
  "list": [
    {
      "id": "5762",
      "name": "",
      "type": 1,
      "first_action": 1460480919000,
      "last_action": 1460480919000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "3",
        "title": "AERO SERVICES <AEROE> IN PACT FOR NOMINATIONS",
        "body": "Aero Services International Inc\nsaid it signed an agreement with Dibo Attar, who controls about\n39 pct of its common stock, under which three nominees to\nAero's board have been selected by Attar.\n    In addition to Attar, the nominees are Stephen L. Peistner,\nchairman and chief executive officer of <McCrory Corp> and\nJames N.C. Moffat III, vice president and secretary of\n<Eastover Corp>.\n Reuter\n\u0003"
      }
    },
    {
      "id": "11348",
      "name": "",
      "type": 1,
      "first_action": 1460480919000,
      "last_action": 1460480919000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "3",
        "title": "GANDALF <GANDF> ACQUIRES STAKE IN DATA/VOICE",
        "body": "Gandalf Technologies Inc said it\nacquired a significant minority equity interest in privately\nheld Data/Voice Solutions Corp, of Newport Beach, Calif., for\nundisclosed terms.\n    Gandalf did not specify the size of the interest.\n    Data/Voice is a three-year-old designer and manufacturer of\na multiprocessor, multiuser MS-DOS computing system that\nGandalf plans to integrate with its private automatic computer\nexchange information system, Gandalf said.\n Reuter\n\u0003"
      }
    },
    {
      "id": "7816",
      "name": "",
      "type": 1,
      "first_action": 1460480919000,
      "last_action": 1460480919000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "3",
        "title": "CELINA <CELNA> SHAREHOLDERS APPROVE SALE",
        "body": "Celina Financial Corp said\nshareholders at a special meeting approved a transaction in\nwhich the company transferred its interest in three insurance\ncompanies to a wholly owned subsidiary which then sold the\nthree companies to an affiliated subsidiary.\n    It said the company's interests in West Virginia Fire and\nCasualty Co, Congregation Insurance co and National Term Life\nInsurance Co had been transferred to First National Indemnity\nCo, which sold the three to Celina Mutual for cash, an office\nbuilding and related real estate.\n Reuter\n\u0003"
      }
    }
  ]
}
{% endhighlight %}

# Troubleshooting

 * No results are returned by the seldon-cli api call above

Check your Kuberentes DNS is working correctly and the reuters-example hostname can be found. Open a bash terminal into the seldon-control container and check using nslookup.:

{% highlight bash %}
kubectl exec -ti seldon-control -- /bin/bash
root@seldon-control:/home/seldon# nslookup reuters-example
Server:				  	   10.0.0.10
Address:				   10.0.0.10#53

Name:					   reuters-example.default.svc.cluster.local
Address: 10.0.0.25
{% endhighlight %}

# Next Steps<a name="next-steps"></a>

 * [Detailed Guide for Content Recomemdation](content-recommendation-guide.html)
