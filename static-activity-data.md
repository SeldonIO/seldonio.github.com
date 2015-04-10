---
layout: default
title: Importing your static data
---

# Importing your static data

After defining your data schema in a previous step, we are now ready to add any previously collected data you have. To add data to the DB it needs to be split into users, items and actions. This data needs to be in CSV format. You should quote any data that has unicode characters in it.

Scripts used for importing your data can be found in the [seldon-server](https://github.com/SeldonIO/seldon-server) project, which you would have cloned in an earlier step.

### Items

Items must conform to the schema with 1 or two extra fields. An example for the schema above is

{% highlight bash %}
id,name,title,artist,genre,price,is_compilation,sales_count
1,"tune1","tune1","artist1","pop",0,1
2,"tune2","tune2","artist2","rock",1,20
3,"tune3","tune3","artist1","rock",1,30
{% endhighlight %}

Note that we have 'id' and 'name' that were not mentioned in the schema. 'id' is a required field for all items in all schemas. It can be any unique string and is your identifier for the item. 'name' is optional and should be a string that you might use to search for the item. A few other things to mention here are

 1. Boolean fields (is_compilation for example) can be 0 or 1, 0 meaning false and 1 meaning true.
 1. Enum fields (genre for example) must be one of the values you defined in the schema
 1. You must provide a header line
 1. There is an example in /your_data/items_data/example_items.csv

Load the CSV into the DB using the `add_items.py` script from the `seldon-server/scripts` folder.
 
### Users
 
Users are much easier as currently it is not possible to specify a schema. So we just need an id and optionally a username:

{% highlight bash %}
id,username
1,phil
2,clive
3,gurminder
4,alex
{% endhighlight %}

Add the CSV file to the DB using the `add_users.py` script in the `seldon-server/scripts` folder.

### Actions

Actions again have no schema to contend with but we need a few extra fields:

{% highlight bash %}
user_id,item_id,value,time
1,2,1,1422128735
2,2,1,1422752450
3,3,1,1422735290
4,1,1,1422792312
5,1,1,1422795111
5,3,1,1422754829
{% endhighlight %}

The first two columns should be obvious. 'value' is a field that represents the magnitude of the action. If all actions are created equal, then you should just set this to one. 'time' is the unixtimestamp of the action.

The actions are not added to the DB, but they require transformation so that the Spark jobs can consume them. Run the `create_actions_json.py` script in the `seldon-server/scripts` folder to create this.

