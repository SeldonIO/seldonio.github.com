---
layout: default
title: Seldon Spark
---

# Introduction
 
The Seldon Spark component is used to run [Apache Spark](https://spark.apache.org/) jobs that process logs and build models to be used by the Seldon Server.

# Set Up

## Prerequisites

To use Seldon Spark, the following need to be installed:

* Java (v = 7)
* Spark (v >= 1.2.0)
* Maven (v >=3)
* Git

## Build the Seldon Spark app jar

{% highlight bash %}
cd ~/
git clone https://github.com/SeldonIO/seldon-spark
cd seldon-spark
git checkout tags/v1.0.1
mvn -DskipTests=true clean package
{% endhighlight %}

# Offline Data Store
The Seldon modelling and data manipulation jobs assume a structure for the data storage. This structure allows easy integration into a production environment where models are created periodically, usually each day. The directory structure is of the form
 {% highlight bash %}
    seldon-models/${CLIENT}/${MODEL}/${DAY}
 {% endhighlight %}
	
e.g. for a matrix_factorization model created for client client1 on 27 Jan 2014 (unix epoch day 16461) would be 

 {% highlight bash %}
    seldon-models/client1/matrix_factorization/16461
 {% endhighlight %}

You can use a network file store, AWS S3 or soon HDFS for the actual store.

The jobs that require activity data will use a start day and a number of days to collect from the filesystem the data they need. They will gather data from folders of the form:

{% highlight bash %}
${input-path}/${client}/actions/start-day
${input-path}/${client}/actions/start-day-1
${input-path}/${client}/actions/start-day-2
.
.
${input-path}/${client}/actions/start-day-(num-days)
{% endhighlight %}

For example:

{% highlight bash %}
/seldon-models/client1/actions/16461
/seldon-models/client1/actions/16460
/seldon-models/client1/actions/16459
{% endhighlight %}

The output path will be of the form:

{% highlight bash %}
${output-path}/${client}/${model}/start-day
{% endhighlight %}

For example:

{% highlight bash %}
s3://seldon-models/client1/matrix-factorization/16461
{% endhighlight %}

# Actions Grouping

To provide the core activity data needed by many modelling jobs we provide a job which collects the raw action data collected by the API and outputs into daily folders for each client. The input format is presently JSON.

{% highlight json %}
    {
	 "client": "myapp",
	 "client_itemid": "http://myapp.com/somepage",
	 "client_userid": "nveof-3242-ssggsvvf",
	 "itemid": 2301,
	 "rectag": "default",
	 "timestamp_utc": "1998-12-02T05:56:13Z",
	 "type": 1,
	 "userid": 69878,
	 "value": 2.0
	}
{% endhighlight %}

The predicitve modelling jobs that utilize activity will read these processed logs in and output models that use the internal user and item ids. The runtime Seldon server will request scoring using these internal ids. The client user and item ids are the client facing friendly ids and can be arbitary strings, e.g. cookie-ids for users and URLs for items.

An example of running the group actions job.  
The options to the job are as follows:  

* **--input-path-pattern** - This reflects the source of the actions and how Seldon Server was setup to store them. The pattern is based on the path to the location, %y for year, %m for month, %d for day.  
* **--input-date-string** - This is the day that will be processed in unix epoch time.
* **--output-path-dir** - This should be the place the other jobs will use as input for their processing.
* **--gzip-output** - Use this option to compress the output data.

{% highlight bash %}
SELDON_SPARK_HOME=~/seldon-spark
DATE_YESTERDAY=$(perl -e 'use POSIX;print strftime "%Y%m%d",localtime time-86400;')
INPUT_DATE_STRING=${DATE_YESTERDAY}
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-1.0.1-jar-with-dependencies.jar
SPARK_HOME=/opt/spark

INPUT_DIR=~/seldon-logs
OUTPUT_DIR=~/seldon-models

${SPARK_HOME}/bin/spark-submit \
    --class "io.seldon.spark.actions.GroupActionsJob" \
    --master local[1] \
    ${JAR_FILE_PATH} \
        --aws-access-key-id "" \
        --aws-secret-access-key "" \
        --input-path-pattern "${INPUT_DIR}/actions.%y/%m%d/*/*" \
        --input-date-string "${INPUT_DATE_STRING}" \
        --output-path-dir "${OUTPUT_DIR}" \
        --gzip-output
{% endhighlight %}


# Matrix Factorization
An algorithm made popular due to its sucess in the Netflix competition. It tries to find a small set of latent user and item factors that explain the user-item interaction data. We use a wrapper around the [Apache Spark ALS](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) implementation.  Note, however, for this release we only provide implicit matrix factorization.

{% highlight bash %}
SELDON_SPARK_HOME=~/seldon-spark
YESTERDAY=$(perl -e 'use POSIX;print strftime "%Y%m%d",localtime time-86400;')
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-1.0.1-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
BASE_DIR=~/seldon-models

NUM_DAYS=120
RANK=30
LAMBDA=0.1
ALPHA=1
ITERATIONS=5
ZOOKEEPER_NODES=zkhost1,zknost2,zkhost3

${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.mllib.MfModelCreation" \
	   --master local[1] \
	   ${JAR_FILE_PATH} \
	   ${CLIENT}
	   ${YESTERDAY}
	   ${NUM_DAYS}
	   ${RANK}
	   ${LAMBDA}
	   ${ALPHA}
	   ${ITERATIONS}
	   ${ZOOKEEPER_NODES}
	   ${BASE_DIR}/${CLIENT}/actions
	   ${BASE_DIR}/${CLIENT}/matrix-factorization
{% endhighlight %}

The various configurable parameters are:

 * client : the name of the client.
 * start-day : the start day as a unix epoch day number to use as input data
 * numdays : the number of days into the past from the start-day to use as input data
 * base_dir : The base folder for input and output 
 * rank : the number of latent factors in the model.
 * iterations : the number of iterations to run the modelling
 * lambda :  the regularization parameter in ALS to stop over-fitting 
 * alpha : governs the baseline confidence in preference observations
 * zookeeper_nodes : the zookeeper nodes to update to inform the seldon-server that a new model is available
 

# Item similarity
Item similarity models find correlations in the user-item interactions to find pairs of items that have consistently been viewed together. The underlying algorithm is the [DIMSUM algorithm in Apache Spark 1.2](https://blog.twitter.com/2014/all-pairs-similarity-via-dimsum).

{% highlight bash %}
SELDON_SPARK_HOME=~/seldon-spark
YESTERDAY=$(perl -e 'use POSIX;print strftime "%Y%m%d",localtime time-86400;')
JAR_FILE_PATH=${SELDON_SPARK_HOME}/target/seldon-spark-1.0.1-jar-with-dependencies.jar
SPARK_HOME=/opt/spark
BASE_DIR=~/seldon-models

${SPARK_HOME}/bin/spark-submit \
	   --class "io.seldon.spark.mllib.SimilarItems" \
	   --master local[1] \
       ${JAR_FILE_PATH} \
	   --client ${CLIENT} \
	   --input-path ${BASE_DIR} \
	   --output-path ${BASE_DIR} \
	   --start-day ${YESTERDAY} \
	   --numdays 180 \
	   --minUsersPerItem 100 \
	   --minItemsPerUser 10 \
	   --dimsum-threshold 0.1 \
	   --maxUsersPerItem 300000 \
	   --limit 30
{% endhighlight %}

The various configurable parameters are:

 * --client : the name of the client.
 * --start-day : the start day as a unix epoch day number to use as input data
 * --numdays : the number of days into the past from the start-day to use as input data
 * --input-path : The base folder for the action input data. It will be expanded to include folders of the form:

{% highlight bash %}
   ${input-path}/${client}/actions/start-day
   ${input-path}/${client}/actions/start-day-1
   ${input-path}/${client}/actions/start-day-2
   .
   .
   ${input-path}/${client}/actions/start-day-(num-days)
{% endhighlight %}

 * --output-path : the base folder for the item similarity output. It will be expanded to place the result of the modeling in:

{% highlight bash %}
 ${output-path}/${client}/item-similarity/start-day
{% endhighlight %}

 * --sample : limit the actions processed to a random subset of all actions. Value 0.0 to 1.0. Default 1.0 (all actions)
 * --limit : return only the top N similarities for each item
 * --dimsum_threshold : A value >= 0 and usually < 1.0. Higher the setting the greater the efficiency but less accurate the result. See the [DIMSUM algorithm](https://blog.twitter.com/2014/all-pairs-similarity-via-dimsum) for further details.
 * --minUsersPerItem : The minimum number of users who need to have interacted with an item for it to be included
 * --minItemsPerUser : The minimum number of items that a user has interacted with for that user to be included
 * --maxUsersPerItem : The maximum number of items a user has interacted with for that user to be included (useful to remove Bots from activity data)
 
Once complete the Seldon database needs to be updated with the similarities so they can be served.

You will need to get from local filesystem, AWS S3 or HDFS the output of the job and run a script to convert to SQL and upload to the Seldon DB. An example for the result stored on the local filesystem is shown below:
{% highlight bash %}
SELDON_SPARK_HOME=~/seldon-spark
CLIENT=client
DB_HOST=db
DB_USER=user
DB_PASS=pass
DAY=16491
cat /seldon-models/${CLIENT}/item-similarity/${DAY}/part* | ${SELDON-SPARK-HOME}/scripts/item-similarity/create-sql > upload.sql
mysql -h${DB_HOST} -u${DB_USER} -p${DB_PASS} ${CLIENT} < upload.sql 
{% endhighlight %}

# Other Spark Based Models

The seldon-spark project also contains other models that can be used. These include:

 * word2vec : Create vector based representations of your products based on the user session activity. This is a novel use of word2vec technology where the "words" are your items (products, articles) and the "sentences" are the session history of users interacting with these items.
 * clustering : For news sites or other sites where item churn is rapid and relevancy of items decays fast we can base recommendation of what similar users are interacting with on your site. These models are based on clustering the users against a static set of clusters (e.g., an item taxonomy) or in a free way (e.g. using fuzzy k-means)
 
