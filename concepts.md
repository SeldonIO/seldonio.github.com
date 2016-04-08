---
layout: default
title: Concepts 
---

# General Seldon Concepts

## **Client**

On each Seldon deployment, it is possible to provide recommendations for several totally different data sets (for example, two websites). These data sets are divided by using different clients. Each call to the API has client as one of the attributes. 

# Content Recommendation Concepts

## **User**

A user of the Seldon Server is any entity that interacts with items and requires recommendations. For example, a person on a news website that is viewing a story and need recommendations on the next story to read.

## **Item**

Items are the things that are to be recommended from. For example, a web link that shows a news story.

## **Action**

Actions are any time when a user interacts with an item (viewing it, buying it, adding it to a cart).

*User, Item and Action can be enriched with arbitrary attributes depending on the use case. For example, a newspaper article might have a subtitle and a category.  When the Seldon Server makes its recommendations for a user, it takes the history of items that he/she has interacted with and their attributes into account.*

# General Prediction Concepts

## **Event**

A event is an arbitrary set of features. These events can be used to build a predictive model to target one feature in the event. For example an event might supply house size and location along with price. A model can be built to predict the price feature from the other features. At runtime a set of features not including price can be supplied and a prediction of the price returned.



 
 
 

