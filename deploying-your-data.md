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

### Users
 
 At the moment, you cannot add extra attributes to a user, so there's no need for a user schema.

### Actions

 At the moment, you can only have one action type and you cannot add extra attributes to an action, so theres no need for an action schema.

## Adding the schema to the DB

Now that we have this schema defined we can add it into the database. This can be acheived by running `add_attr_schema.py` in the [seldon server scripts](https://github.com/SeldonIO/seldon-server/tree/master/scripts) folder. If successful, you will receive a dimensions.json file in your folder you ran the script from.  

