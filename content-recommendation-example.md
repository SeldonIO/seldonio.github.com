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
kubernetes/bin/run_recommendation_microservice.sh reuters-example seldonio/reuters-example 1.0 reuters
{% endhighlight %}

The script will create the Kubernetes Deployment and use the seldon-cli to update the "reuters" client to add the microservice as a runtime algorithm. 

# Serve Recommendations<a name="recommendations"></a>
You can now call the seldon server using the client details for the reuters client to get predictions. We have provided an example script to do this in ```kubernetes/examples/reuters/get_reuters_recommendation.sh``` which provides recommendations based on an existing article. This example script assumes you have started kuberentes locally and the server is accessible at port 30000. You will need to [change the endpoint](install.html#endpoint) to the location of the seldon server exposed via Kubernetes if not. Running this should produce an output like:


{% highlight json %}
{
  "size": 5,
  "requested": 5,
  "list": [
    {
      "id": "21576",
      "name": "",
      "type": 1,
      "first_action": 1460104099000,
      "last_action": 1460104099000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "2",
        "title": "SIX KILLED IN SOUTH AFRICAN GOLD MINE ACCIDENT",
        "body": "Six black miners have been killed\nand two injured in a rock fall three km underground at a South\nAfrican gold mine, the owners said on Sunday.\n    <Rand Mines Properties Ltd>, one of South Africa's big six\nmining companies, said in a statement that the accident\noccurred on Saturday morning at the <East Rand Proprietary\nMines Ltd> mine at Boksburg, 25 km east of Johannesburg.\n    A company spokesman could not elaborate on the short\nstatement.\n REUTER\n\u0003"
      }
    },
    {
      "id": "21575",
      "name": "",
      "type": 1,
      "first_action": 1460104099000,
      "last_action": 1460104099000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "2",
        "title": "SOVIET INDUSTRIAL GROWTH/TRADE SLOWER IN 1987",
        "body": "The Soviet Union's industrial output is\ngrowing at a slower pace in 1987 than in 1986 and foreign trade\nhas fallen, Central Statistical Office figures show.\n    Figures in the Communist Party newspaper Pravda show\nindustrial production rose 3.6 pct in the first nine months of\n1987 against 5.2 pct in the same 1986 period.\n    Foreign trade in the same period fell 3.6 pct from the 1986\nperiod as exports fell by 0.5 pct and imports dropped by 4.2\npct.\n    Foreign trade in the nine months totalled 94.2 billion\nroubles. Separate import and export figures were not given.\n    One factor affecting industrial growth was the introduction\nof a new quality control plan, Western economists said. Last\nyear's calculations of industrial output included all goods,\nirrespective of quality.\n    Under the new plan, introduced in line with Soviet leader\nMikhail Gorbachev's drive to modernise the economy, special\ninspectors have the right to reject goods they consider below\nstandard.\n    Pravda said 42 mln roubles worth of defective goods were\nrejected in the nine-month period.\n    The figures also showed that on October 1, there were more\nthan 8,000 cooperative enterprises employing over 80,000\npeople. More than 200,000 were employed in the private sector,\nPravda said, without giving comparative figures.\n    The promotion of the cooperative and private sectors of the\neconomy has been an important part of the modernisation\ncampaign, with measures introduced recently to allow the\nsetting up of small shops on a private basis.\n    Labour productivity rose 3.7 pct in the first nine months\nagainst 4.8 pct growth in January to September 1986.\n    But western economists said they treat Soviet productivity\nfigures with caution, as they are more broadly based than in\nthe West, which measures worker output over a given period.\n    Pravda said there were 283.8 mln people in the Soviet Union\nas of October 1. In the January to September 1987 period 118.5\nmln people were employed, a rise of 4.4 pct on the same period\nlast year. Average earnings were 200 roubles a month against\n194 roubles a year ago.\n REUTER\n\u0003"
      }
    },
    {
      "id": "21574",
      "name": "",
      "type": 1,
      "first_action": 1460104099000,
      "last_action": 1460104099000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "2",
        "title": "JAPAN/INDIA CONFERENCE CUTS GULF WAR RISK CHARGES",
        "body": "The Japan/India-Pakistan-Gulf/Japan\nshipping conference said it would cut the extra risk insurance\nsurcharges on shipments to Iranian and Iraqi ports to a minimum\nthree pct from 4.5 pct on October 25.\n    It said surcharges on shipments of all break-bulk cargoes\nto non-Iraqi Arab ports would be reduced to 3.0 pct from 4.5.\n    A conference spokesman declined to say why the move was\ntaken at a time of heightened tension in the Gulf.\n REUTER\n\u0003"
      }
    },
    {
      "id": "21573",
      "name": "",
      "type": 1,
      "first_action": 1460104099000,
      "last_action": 1460104099000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "2",
        "title": "TOKYO DEALERS SEE DOLLAR POISED TO BREACH 140 YEN",
        "body": "Tokyo's foreign exchange market is watching\nnervously to see if the U.S. Dollar will drop below the\nsignificant 140.00 yen level, dealers said.\n    \"The 140 yen level is key for the dollar because it is\nconsidered to be the lower end of the reference range. If the\ncurrency breaks through this level, it may decline sharply,\"\nsaid Hirozumi Tanaka, assistant general manager at Dai-ichi\nKangyo Bank Ltd's international treasury division.\n    The dollar was at 141.10 yen at midday against Friday\ncloses of 142.35/45 in New York and 141.35 here.\n    The dollar opened at 140.95 yen and fell to a low of\n140.40. It was 1.7733/38 marks against 1.7975/85 in New York\nand 1.8008/13 here on Friday, after an opening 1.7700/10.\n    The currency's decline was due to remarks on Sunday by U.S.\nTreasury Secretary James Baker, dealers said.\n    \"The dollar fell over the weekend on increased bearish\nsentiment after Baker's comments,\" said Dai-ichi's Tanaka. He\nsaid this stemmed from mounting concern that cooperation among\nthe group of seven (G-7) industrial nations to implement the\nLouvre accord to stabilise currencies might be fraying.\n    The dollar's fall was also prompted by a record one-day\ndrop in the Dow Jones industrial average on Friday and weakness\nin U.S. Bond prices, dealers said.\n    Baker said the Louvre accord was still operative but he\nstrongly criticised West German moves to raise key interest\nrates. Operators took Baker's comment to indicate impatience\nwith some G-7 members for failing to stick to the Louvre accord\ndue to their fears of increasing inflation.\n    Rises in interest rates aimed at dampening inflationary\npressures also slow domestic demand.\n    West Germany and Japan had both pledged at G-7 meetings to\nboost domestic demand to help narrow the huge U.S. Trade\ndeficit, Tanaka said.\n    U.S. August trade data showed the U.S. Deficit at a still\nmassive 15.68 billion dlrs. But if West Germany raises interest\nrates, this would run counter to the pledge, he said.\n    \"Operators are now waiting to see if the G-7 nations\ncoordinate dollar buying intervention,\" said Soichi Hirabayashi,\ndeputy general manager of Fuju Bank Ltd's foreign exchange\ndepartment.\n    The target range set by the Louvre accord is generally\nconsidered to be 140.00 to 160.00 yen, dealers said.\n    \"The market is likely to try the 140 yen level in the near\nfuture and at that time, if operators see the G-7 nations\nfailing to coordinate intervention, they would see the Louvre\naccord as abandoned and push the dollar down aggressively,\"\nHirabayashi said. He said the U.S. Currency could fall as low\nas 135 yen soon.\n REUTER\n\u0003"
      }
    },
    {
      "id": "21571",
      "name": "",
      "type": 1,
      "first_action": 1460104099000,
      "last_action": 1460104099000,
      "popular": false,
      "demographics": [],
      "attributes": {},
      "attributesName": {
        "recommendationUuid": "2",
        "title": "N.Z.'S CHASE CORP MAKES OFFER FOR ENTREGROWTH",
        "body": "Chase Corp Ltd <CHCA.WE> said it will\nmake an offer for all fully-paid shares and options of\n<Entregrowth International Ltd> it does not already own.\n    Chase, a property investment firm, said it holds 48 pct of\nEntregrowth, its vehicle for expansion in North America.\n    It said agreements are being concluded to give it a\nbeneficial 72.4 pct interest.\n    The offer for the remaining shares is one Chase share for\nevery three Entregrowth shares and one Chase option for every\nfour Entregrowth options. Chase shares closed on Friday at 4.41\ndlrs and the options at 2.38.\n    Entregrowth closed at 1.35 dlrs and options at 55 cents.\n    Chase said the offer for the remaining 27.6 pct of\nEntregrowth, worth 34.2 mln dlrs, involved the issue of 5.80\nmln Chase shares and 3.10 mln Chase options.\n    Chase chairman Colin Reynolds said the takeover would allow\nEntregrowth to concentrate on North American operations with\naccess to Chase's international funding base and a stronger\nexecutive team. He said there also would be benefits from\nintegrating New Zealand investment activities.\n    Chase said the offer is conditional it receiving accptances\nfor at least 90 pct of the shares and options.\n REUTER\n\u0003"
      }
    }
  ]
}
{% endhighlight %}

