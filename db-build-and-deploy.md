---
layout: default
title: Introduction
---

Start with a fresh MySQL installation. Clone the [seldon-server]() project. In it there are the schemas to add that you need. **NOTE** when referring to 'databases' below it means that MySQL concept of databases rather than a database server.

* Create a database called 'api'.
* Run the api.sql file in that database.
* Create two entries in the consumer table like the below

{% highlight text %}
+--------------+-----------------+-------------+------------+---------------------+------+--------+--------+-------+
| consumer_key | consumer_secret | name        | short_name | time                | url  | active | secure | scope |
+--------------+-----------------+-------------+------------+---------------------+------+--------+--------+-------+
| consumerkey  |                 | test client | testclient | 2013-07-12 14:16:00 | NULL |      1 |      0 | js    |
| consumerkey1 | consumersecret1 | test client | testclient | 2014-00-00 00:00:00 | NULL |      1 |      0 | all   |
+--------------+-----------------+-------------+------------+---------------------+------+--------+--------+-------+
{% endhighlight %}

These are the entries that control access to the REST (scope=all) and JS API (scope=js). `name` is a long name with spaces and `short_name` is how you will refer to this **client** from now on. The keys and secrets should be generated.

* Create a database called whatever the `short_name` for your client was. 
* Run the client.sql file in that database.
* Done!
