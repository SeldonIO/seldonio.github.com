---
layout: default
title: Using your own data
---

The Seldon Recommendation Engine allows a wide variety of data to be recommended from. We will introduce the restrictions below and then talk about how to get your historical data generating recommendations!

# Key concepts

Seldon recognises three key pieces of data: Users, Items and Actions.

Users are what they sound like : any entity that interacts with items and requires recommendations.

Items are the things that are to be recommended from.

Actions are any time when a user interacts with an item (viewing it, buying it, adding it to a cart)

Each of the above data pieces can be enriched with attributes (see below). When the Seldon Recommendation Engine makes its recommendations for a user, it takes the history of items that he/she has interacted with and their attributes into account.

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

### Users

 At the moment, you cannot add extra attributes to a user, so theres no need for a user schema.

### Actions

 At the moment, you can only have one action type and you cannot add extra attributes to an action, so theres no need for an action schema.

## Adding the schema to the DB

Now that we have this schema defined we can add it into the database. This can be acheived by using one of the shell scripts defined in the /your_data directory. Put your schema in /your_data/schema/ (you will see the example schema in there -- example_schema.javascript) then run the following.

{% highlight bash %}
    ./load_schema.sh test1 example_schema.javascript
{% endhighlight %}

replacing 'test1' with the DB that you want to use and 'example_schema.javascript' with the schema file that you want (just the filename, not the folder).
{% highlight javascript %}
    We've added a number of databases for you to use. Take your pick from test1 -- test10
{% endhighlight %}

## The actual data

To add data to the DB it needs to be split into users, items and actions. This data needs to be in CSV format. You should quote any data that has unicode characters in it.

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
Once you have this CSV for your data, save it to /your_data/items_data.

Load it into the DB using
{% highlight bash %}
    ./load_items.sh test1 example_items.csv
{% endhighlight %}

### Users

Users are much easier as currently it is not possible to specify a schema. So we just need an id and optionally a username:

{% highlight bash %}
id,username
1,phil
2,clive
3,gurminder
4,alex
{% endhighlight %}

Put the CSV in /your_data/users_data/ and then run
{% highlight bash %}
    ./load_users.sh test1 example_users.csv
{% endhighlight %}

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

