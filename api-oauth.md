---
layout: default
title: OAuth API
---

The Seldon API interacts with three main concepts:

* [Items](#items) : Pieces of content, e.g., articles, products, game items
* [Users](#users) : Users of a service
* [Actions](#actions) : Actions done by users on items, e.g., read an article, purchase a product or pick up a game item

Finally, one then uses the API to provide predictions, presently:

* [Recommendations](#recommendations)

An API method can be called with the following template
{% highlight http %}
[GET|POST]      /endpoint?oauth_token=t
{% endhighlight %}
	
For security reasons using the HTTPS protocol is recommended.

# API Input Endpoints

This section outlines the endpoints that allow you to input data the API needs to support the personalisation and recommendation methods.


## Items <a name="items"></a>


An item is an element within the service identified by id and name. An item generally maps to a piece of content that users interact with.

A service may have different pre-defined item types, and for each item type a set of attributes can be specified.

{% highlight http %}
POST   /items
{% endhighlight %}

Input 

* id : required string
* type : (optional) int
* name : (optional) string
* attributes : (optional) map[int,int] the keys of which should be the attribute id and the values the attribute value id
* attributesName : (optional) map[string,string] the keys of which should be the attribute name and the values the attribute value name. It is an alternative to attributes.

Example

The service is an E-commerce website selling music and video games.

Item types:

* Music                 [type 1]
* Video games         [type 2] 


The two item types have different attributes:

Music Item Attributes:

* string Title                         [attr_id 1]
* string Artist                 [attr_id 2]
* enum Genre                 [attr_id 3]
* double Price                 [attr_id 4]
where
* Genre is the enumeration: 

(pop [value_id 1], rock [value_id 2], rap [value_id 3])


Video game Item Attributes:

   * string Name                 [attr_id 5]
   * enum Category                 [attr_id 6]
   * enum Console                [attr_id 7]
   * double Price                 [attr_id 8]

where:

   * Category is the enumeration: 
(arcade [value_id 1], sport [value_id 2], action [value_id 3])
   * Console is the enumeration: 
(wii [value_id 1], ps3 [value_id 2], xbox [value_id 3])

The JSON to post a music item [‘xxx’,‘yyy’,‘rock’,12.99] having id ‘abc’ using the attributeName map:

{% highlight json %}
{
    "id" : "abc",
    "type" : 1,
    "name" : "xxx",
    "attributesName" : { 
    		     "title"        :   "xxx",
		     "artist"        :   "yyy",
		     "genre"        :   "rock",
		     "price"        :   12.99 
		     }    
}
{% endhighlight %}

The method to add a video game item [‘www’,‘sport’,‘wii’,29.99] having id ‘def’ using the attribute map is:
{% highlight json %}
{
    "id" : "def",
    "type" : 2,
    "name" : "www"
    "attributes” : {
                        5        :   "www",
			6        :   2,
			7        :   1,
			8        :   29.99
		}    
}
{% endhighlight %}

## Users <a name="users"></a>
A user is a service customer identified by an id and a username.
A set of user attributes (demographics) can be defined.

{% highlight http %}
POST     /users
{% endhighlight %}	

Input 

* id required string
* username (optional) string
* active (optional) boolean
* attributes (optional) map[int,int] the keys of which should be the attribute id and the values the attribute value id.
* attributesName (optional) map[string,string] the keys of which should be the attribute name and the values the attribute value name. It is an alternative to attributes.

Example

The service is a Social Network website.


The user attributes are: 

* enum Gender         [attr_id 1]
* int Age                [attr_id 2]
* enum Language         [attr_id 3]
* string Location         [attr_id 4]


where:

* Gender is the enumeration (male [value_id 1], female [value_id 2])
* Language is the enumeration (en [value_id 1], it [value_id 2], es [value_id 3])

The JSON to post a user [‘male’,29,‘es’,‘New York, US’] having id ‘123’ and username ‘xxx’ using the attribute map:
{% highlight json %}
{
    "id" : "123",
    "username" : "xxx"
    "active" : true,
    "attributes": {
              	  1        :        1,
		  2        :        29,
		  3        :        3,
		  4        :        "New York,US"
		  }    
}
{% endhighlight %}	

and using the attributesName map:
{% highlight json %}
{
    "id" : "123",
    "username" : "xxx"
    "active" : true,
    "attributesName" : {
                        "Gender"        :        "male",
			"Age"                :        29,
			"Language"        :        "es",
			"Location"        :        "New York,US"
			}    
}
{% endhighlight %}	

## Actions <a name="actions"></a>
An action is any interaction of a user with an item within the service.
A service may have different action type and may specify further optional information as value, comment and tags.

{% highlight http %}
POST     /actions
{% endhighlight %}

Input

* user (required) string identifying the user
* item (required) string identifying the item
* type (required) int specifying the action type
* value (optional) int specifying the action intensity
* comment (optional) string attached to the action
* tags (optional) string comma separated attached to the action


Example

The service is an E-commerce website.

The action types are:

* click                [action_id 1] (no optional param)
* page view                [action_id 2] (value = duration in seconds of the action)
* review                [action_id 3] (value = rating, comment = review text)
* place in basket        [action_id 4] (no optional param)
* purchase                [action_id 5] (value = price)


The JSON to post a click action performed by user “xxx” on the item “yyy”:

{% highlight json %}
{
"user" : "xxx",
"item" : "yyy",
"type" : 1
}
{% endhighlight %}	

The JSON to post a page view action performed by the user “xxx” on the item “yyy” that lasted 40 seconds:

{% highlight json %}
{
"user" : "xxx",
"item" : "yyy",
"type" : 2,
"value": 40
}
{% endhighlight %}	

The JSON to  post a review action performed by the user “xxx” on the item “yyy” with rating 4 and comment “...”:

{% highlight json %}
{
"user"    :        "xxx",
"item"    :        "yyy",
"type"    :        2,
"value"   :        4,
"comment" : "..."
}
{% endhighlight %}


## API Personalisation Endpoints <a name="recommendations"></a>

This section describes the endpoints which provide personalisation and recommendation functionality derived from the user, item, action data that has been POSTed to the API.


### Recommended Items

A list of recommended items can be retrieved for a user. The list size can be specified with a limit. The filter type, for selecting only a specific item type, or dimension, for selecting items in a specific category can be applied.
If the service has provided textual content for the items, a personalized search can be performed specifying the keyword parameter. The relevant items matching the keywords are retrieved and reordered for the specified user. 

Scenario

This method could be used as part of the home page for a user to provide links to pages for items they may have interest in. The keyword parameter can be used to provide a personalised search engine for the service.

{% highlight http %}
GET     /users/{user_id}/recommendations
{% endhighlight %}	

Parameters
     
* limit (optional) int size of the recommendations list (by default set to 10)
* type (optional) int filtering on the item type  
* dimension (optional) int 
** if dimension = -1 retrieves recommendations using trust graph
** if dimension = 0 retrieves recommendations using trust and user preferences (default)
** if dimension > 0 filters recommendations on a specific item category
* keyword (optional) string comma separated containing the keywords for the search

Example

The API call to get the top 20 recommend items for the user ‘xxx’: 

{% highlight http %}
GET /users/xxx/recommendations?limit=20
{% endhighlight %}	

The API call to get the top 10 recommended items in the dimension 3 (for example “genre=rock”):

{% highlight http %}
GET /users/xxx/recommendations?dimension=3
{% endhighlight %}	

Output

{% highlight json %}
{
  "list" :[
           {"item"        :        "x", "pos"                :        "1"},
	   {"item"        :        "y", "pos"                :        "2"}
	   ]    
}
{% endhighlight %}	

where list is an array containing a list of items. For each item the pos specify the rank in the recommendation list

## Appendix

### Dimensions
A dimension is a subset of the item set based on the item attributes.

Each item attribute-value generates a dimension containing all the items having the specific attribute-value. 

Dimensions allow the item space to be divided into relevant semantic sections which can then be used to characterize users’ preferences and behaviour (for example dimensions can be used to spot a user’s interest for specifics topics and categories, price range, content language and so forth).


Example

The service is an E-commerce Music Website. The item set is composed of songs.


The item attribute definition is:
     
* string name        [attr_id 1]
* string artist                [attr_id 2]
* enum category        [attr_id 3]
* double price        [attr_id 4]


Where:

* category is the enumeration  
** (pop [value_id 1], rock [value_id 2], rap [value_id 3])
* a range definition is created for the price 
** (<10 [value_id 1], 10-20 [value_id 2], >20 [value_id 3])


We’ll have the following dimension definition:
   
* dimension1 [dim_id 1, attr_id 3, value_id 1] (category = pop)
* dimension2 [dim_id 2, attr_id 3, value_id 2] (category = rock)
* dimension3 [dim_id 3, attr_id 3, value_id 3] (category = rap)
* dimension4 [dim_id 4, attr_id 4, value_id 1] (category =<10)
* dimension5 [dim_id 5, attr_id 4, value_id 2] (category = 10-20)
* dimension6 [dim_id 6, attr_id 4, value_id 3] (category => 20)


This item would be in the dimension2 and dimension5:

* name = xxx
* artist = yyy
* category = rock
* price = 19.99


### Demographics
A demographic is a subset of the user set based on the user attributes.

Each user attribute-value generates a demographic containing all the users having the specific attribute-value. 
Demographics allow the user space to be divided into useful semantic sections which can help in clustering items for classes of users and provide demographic-based recommendations for new users.


Example

The service is a Social Network.

The user attribute definition is:
         
* string username          [attr_id 1]
* enum gender                 [attr_id 2]
* int age          [attr_id  3]


where:

* gender is the enumeration
**(female [value_id 1], male [value_id 2])
* a range definition is created for the age 
** (<18 years [value_id 1], 19-40 years [value_id 2], >41 years [value_id 3])


We might have the following demographic definition:
            
* demographic1 [demo_id 1, attr_id 2, value_id 1] (category = female)
* demographic2 [demo_id 2, attr_id 2, value_id 2] (category = male)
* demographic3 [demo_id 3, attr_id 3, value_id 1] (category =< 18 years)
* demographic4 [demo_id 4, attr_id 3, value_id 2] (category = 19-40 years)
* demographic5 [demo_id 5, attr_id 3, value_id 3] (category => 41 year)


Note that the attribute username is not used for the demographics creation since it’s not a property that could classify the user set.


This user would be in the dimension2 and dimension4:

* username = xxx
* gender = male
* age = 23 years

## Best Practices
There are a set of best practices we recommend when using the Seldon API:
           
* Provide both curated content and recommendations
** Try to mix curated content links with recommended links so a user of the site has opportunities to explore new content as well as have a set of personalised links provided by the API. This allows users to explore areas and subjects they may not have expressed opinions about in the past, and therefore helps the API provide better personalisations in future.
* Cache API responses
** Where possible, cache API responses for a limited time so as not to call the API when 
the cached response is appropriate. Seldon can provide advice on appropriate 
caching times for each endpoint given the particular user and content activity of your 
site.
* Prefetch data
** Whenever it is possible, call the API methods trying to predict the information the service is likely to need in advance. In this way page load time will be minimized.

