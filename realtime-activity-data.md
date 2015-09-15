---
layout: default
title: Real Data Injestion
---

##### Content Recommendation Steps

[concepts](/concepts.html) --> [setup server](/seldon-server-setup.html) --> [logging](/seldon-logging.html) --> [configure data](/item-recommendation-data.html) --> **realtime activity** --> [offline model](/offline-models.html) --> [runtime configuration](/runtime-recommendation.html) --> [microservices](pluggable-recommendation-algorithms.html) --> [recommendations](api.html)

# Realtime data injestion

For production setting the REST API should be used to inject activity data into Seldon. You can use:

 * The server to server [Oauth REST API](/api-oauth.html) using the ```/actions``` endpoint
 * The [javascript REST API](/api-javascript.html) using the ```/js/actions``` endpoint

## Considerations

 * For activity based models a large amount of data is usually needed. You may therefore wish to activiate real time data injestion for some days before activating the recommendations






