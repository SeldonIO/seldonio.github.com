---
layout: default
title: Item Recommendation Data
---

##### Content Recommendation Steps

[concepts](/concepts.html) --> [setup server](/seldon-server-setup.html) --> [logging](/seldon-logging.html) --> **configure data** --> [realtime activity](/realtime-activity-data.html) --> [offline model](/offline-models.html) --> [runtime configuration](/runtime-recommendation.html) --> [microservices](pluggable-recommendation-algorithms.html) --> [recommendations](api.html)

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

## Adding the schema to the DB

Now that we have this JSON schema defined we can add it into the database.
This can be achieved using [Seldon Shell](/seldon-shell.html).

Use the [attr](/seldon-shell.html#attr) command to edit and apply the json schema.

# Importing your static data

After defining your data schema in a previous step, we are now ready to add any previously collected data you have. To add data to the DB it needs to be split into users, items and actions. This data needs to be in CSV format. You should quote any data that has unicode characters in it.

The [Seldon Shell](/seldon-shell.html) can be used for importing your data.

### Items

Items must conform to the schema with 1 or two extra fields. An example for the schema above is

{% highlight bash %}
id,name,title,artist,genre,price,is_compilation,sales_count
1,"tune1","tune1","artist1","pop",0,1
2,"tune2","tune2","artist2","rock",1,20
3,"tune3","tune3","artist1","rock",1,30
{% endhighlight %}

Note that we have '**id**' and '**name**' that were not mentioned in the schema. 'id' is a required field for all items in all schemas. It can be any unique string and is your identifier for the item. 'name' is optional and should be a string that you might use to search for the item. A few other things to mention here are

 1. Boolean fields (is_compilation for example) can be 0 or 1, 0 meaning false and 1 meaning true.
 1. Enum fields (genre for example) must be one of the values you defined in the schema
 1. You must provide a header line
 1. There is an example in /your_data/items_data/example_items.csv

Load the CSV into the DB using the Seldon Shell [import items](/seldon-shell.html#import) command.
 
### Users
 
Users are much easier as currently it is not possible to specify a schema. So we just need an **id** and optionally a username:

{% highlight bash %}
id,username
1,phil
2,clive
3,gurminder
4,alex
{% endhighlight %}

Load the CSV into the DB using the Seldon Shell [import users](/seldon-shell.html#import) command.

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

The actions are not added to the DB, but they require transformation so that the Spark jobs can consume them.

Use the Seldon Shell [import actions](/seldon-shell.html#import) command to create this.

Note: When when importing actions this way, it`s not necessary to run the group actions job - however the output file needs to be in the right place for the other offline jobs to find it.
When using the Seldon Shell, update seldon.conf file if necessary with the value for "seldon_models" directory.
As an example when using the defaults and a client called "movielens", the created file would appear as follows:

{% highlight bash %}
~/.seldon/seldon-models/movielens/actions/1/actions.json
{% endhighlight %}

