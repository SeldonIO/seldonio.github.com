---
layout: default
title: Configure your data attributes
---

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

