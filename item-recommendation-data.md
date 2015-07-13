---
layout: default
title: Item Recommendation Data
---

This section will show you how to define your item meta data and import any historical actions you may have.

# Configure your data attributes

The Seldon Server allows a wide variety of data to be recommended from. To support all these types we allow you define the schema that your data will fit into. We will introduce some restrictions to this below and then talk about how to get your schema uploaded.

# Restrictions

For now the following restrictions are in place:

 1. You cannot associate extra attributes with users.
 2. You cannot have more than one user, item or action type.

# Your data

## Data schema
 
The Seldon database needs to know what format your data will be in when you pass it, so the first thing that needs to be set in place is a schema file for each data point. This is defined in a JSON format and passed to the scripts that import data.

### Items
 
Here is an example schema for an item representing a music album:

{% highlight javascript %}
	{
    "types": [{
            "type_id": 1,
            "type_name": "music",
            "type_attrs": [
                {"name":"title","value_type":"string"},
                {"name":"artist","value_type":"string"},
                {"name":"genre","value_type":["pop","rock","rap"]},
                {"name":"price","value_type":"double"},
                {"name":"is_compilation","value_type":"boolean"},
                {"name":"sales_count", "value_type":"int"}
                ]
            }
            ]
	}
{% endhighlight %}

Let's go though these fields one by one

 1. type_id: Distinguishes between different types of items for example movies and music. For now, Seldon only allows one item type so this must always be 1
 1. type_name: unique name for this type of item
 1. type_attrs: a list of attributes that can be associated with this item type
 1. type_attrs -> name: the name of the attribute in question
 1. type_attrs -> value_type: what type of data to expect for this attribute. Valid values are 'string', 'double', 'datetime', 'boolean', 'int' and a list. The list is a special case where the data can be one of a restricted list (an enum essentially)

 Once you have worked out how to put your data schema in this format, write the JSON to a file and save it for later.

## Adding the schema to the DB

Now that we have this schema defined we can add it into the database. This can be acheived by running `add_attr_schema.py` in the `seldon-server/scripts` folder. For example, assuming a client client1 and example db settings:

{% highlight bash %}
python add_attr_schema.py -db-host localhost -db-user user1 -db-pass mypass -client client1 -schema-file item_attr.json
{% endhighlight %}

# Historical data
The `seldon-server/scripts` folder also contains scripts to prepopulate and process historical data.

## Add items with their meta data
If we have a CSV with items and their meta data it can be processed with the `add_items.py` script. One column should be `id` for the item id and other columns can be as we specified in the `item_attr_schema.json`.

{% highlight bash %}
python add_items.py -db-host localhost -db-user user1 -db-pass mypass -client client1 -items items.csv
{% endhighlight %}

### Users
 
 At the moment, you cannot add extra attributes to a user, so there's no need for a user schema. But if you have a list of existing user ids you can add them with:

{% highlight bash %}
python add_users.py -db-host localhost -db-user user1 -db-pass mypass -client client1 -users users.csv
{% endhighlight %}

### Actions

 At the moment, you can only have one action type and you cannot add extra attributes to an action, so theres no need for an action schema. if you have a list of actions in csv format they can be processed. The expected fields are:
 
 * `user_id` : the user id : should correspond to user_ids added above
 * `item_id` : the item id : should correspond to item_ids added above
 * `value` : a value associated with the action for example the rating
 * `time` : a unix timestamp

You can then run

{% highlight bash %}
python create_actions_json.py -db-host localhost -db-user user1 -db-pass mypass -client client -actions actions.csv -out actions.json
{% endhighlight %}


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

Note: When using the `create_actions_json.py`, it`s not necessary to run the group actions job - however the  the output file needs to in the right place for the other offline jobs to find it.
{% highlight bash %}
${SELDON_MODELS_DIR}/${CLIENT}/actions/${DAY}/actions.json
eg.
~/seldon-models/movielens/actions/1/actions.json
{% endhighlight %}

