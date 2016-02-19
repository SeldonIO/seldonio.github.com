---
layout: default
title: Seldon Fluentd Configuration
---

# Seldon Fluentd Configuration
When running multiple Seldon servers at scale we need to stream the activity logs to a central location such as AWS S3, HDFS or a central file store. We have found Fluentd from Treasure Data is idle for this. We will discuss here what you need to do to set this up.

You will need to install td-agent which is a packaged version of Fluentd. See [here](http://docs.fluentd.org/v0.12/categories/installation).

Seldon server has four log files:

 * "restapi.log" : all REST API calls
 * "ctr.log" : Impressions and Clicks for recommendations
 * "ctr-alg.log" : Detailed Impressions and Clicks for recommendations with algorithms, and returned items
 * "actions.log" : All action calls

If you have several Seldon servers running you would want to collect these logs and transfer them to a central location. The td-agent.conf configuration file for this can be found in ```seldon-server/td-agent/seldon-server-td-agent.conf.base.example``` and is shown below. You should replace ```<TOMCAT_HOME>``` with the location of your Apache tomcat home folder:

{% highlight text %}
<source>
  type tail
  format /^(?<time>[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+)\,(?<retval>[0-9]+)\,(?<consumer>[^\,]+)\,(?<httpmethod>[^\,]+)\,(?<servlet>[^\,]+)\,(?<path>[^\,]+)\,(?<query>[^\,]+)\,(?<exectime>[^\,]+)\,(?<uuid>[^\,]+)(\,(?<bean>[^\,]+))?(\,(?<algorithm>[^\,]+))?$/
  time_format %Y,%m,%d,%H,%M,%S
  path <TOMCAT_HOME>/logs/seldon-server/base/restapi.log
  tag restapi.calls
  pos_file /var/log/td-agent/tailPos
</source>

<source>
  type tail
  format /^(?<time>[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+)\,(?<click>[^\,]+)\,(?<consumer>[^\,]+)\,(?<user>[^\,]+)\,(?<item>[^\,]+)\,(?<rectag>[^\,]+)$/
  time_format %Y,%m,%d,%H,%M,%S
  path <TOMCAT_HOME>/logs/seldon-server/base/ctr.log
  tag restapi.ctr
  pos_file /var/log/td-agent/ctrTailPos
</source>


<source>
  type tail
  format /^(?<time>[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+)\,(?<click>[^\,]+)\,(?<consumer>[^\,]+)\,(?<alg>[^\,]+)\,(?<pos>[^\,]+)\,(?<userid>[^\,]+)\,(?<useruuid>[^\,]+)\,(?<itemid>[^\,]+)\,(?<actions>[^\,]+),(?<recommendations>[^\,]+)?,(?<abkey>[^\,]+)?\,(?<rectag>[^\,]+)?$/
  time_format %Y,%m,%d,%H,%M,%S
  path <TOMCAT_HOME>/logs/seldon-server/base/ctr-alg.log
  tag restapi.ctralg
  pos_file /var/log/td-agent/ctrAlgTailPos
</source>


<source>
  type tail
  format /^(?<time>[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+\,[0-9]+)\,(?<client>[^\,]+)\,(?<rectag>[^\,]+)\,(?<userid>[0-9]+)\,(?<itemid>[0-9]+)\,(?<type>[0-9]+)\,(?<value>[0-9\.]+)\,\"(?<client_userid>[^(\")]+)\"\,\"(?<client_itemid>[^(\")]+)\"$/
  time_format %Y,%m,%d,%H,%M,%S
  types userid:integer,itemid:integer,type:integer
  path <TOMCAT_HOME>/logs/seldon-server/actions/actions.log
  tag actions.live
  pos_file /var/log/td-agent/actionsAccessPos
</source>

<match restapi.**>
  type forward
  buffer_type file
  buffer_path /var/log/td-agent/logging.*.buffer
  hard_timeout 20m
  expire_dns_cache 1m
  <server>
    name td-agent1
    host tdagent1
    port 24224
    weight 60
  </server>
  <server>
    name td-agent2
    host tdagent2
    port 24224
    weight 60
    standby
  </server>
</match>

<match actions.**>
  type forward
  buffer_type file
  buffer_path /var/log/td-agent/actions.*.buffer
  hard_timeout 20m
  expire_dns_cache 1m
  <server>
    name td-agent1
    host tdagent1
    port 24224
    weight 60
  </server>
  <server>
    name td-agent2
    host tdagent2
    port 24224
    weight 60
    standby
  </server>
</match>

{% endhighlight %}

 The above config does the following:

  * tails the 4 core log files
  * forwards the logs to one of two central td-agent servers called "tdagent1" and a reserve server "tdagent2".

At the central td-agent server you will need to set the config to forward the logs to a central backing store such as AWS S3, or HDFS.
An example central config can be found at ```seldon-server/fluentd/seldon-td-agent.conf.central.example``` and shown below, which forwards to AWS S3 and Kafka the restapi logs:

{% highlight text %}
<source>
  type forward
</source>

<match restapi.**>
  type copy  
  <store>
          type                kafka
          brokers             127.0.0.1:9092
          default_topic       seldon_ctr
          output_data_type    json
          output_include_tag  true
          output_include_time true
          max_send_retries    3
          required_acks       0
          ack_timeout_ms      1500
  </store>
  <store>
          type s3

          aws_key_id AWS_KEY
          aws_sec_key AWS_SECRET
          s3_bucket MYBUCKET
          s3_endpoint s3-eu-west-1.amazonaws.com
          path fluentd/
          buffer_type file
          buffer_path /var/log/td-agent/s3.*.buffer
          time_slice_format %Y/%m%d/%H/%Y%m%d-%H
          s3_object_key_format %{path}%{time_slice}_%{index}_%{hostname}_misc.%{file_extension}
          flush_interval 60s
          utc
  </store>

</match>
{% endhighlight %}

You should check the full docs for the [S3](http://docs.fluentd.org/articles/out_s3) and [kafka](https://github.com/htgc/fluent-plugin-kafka/). The kafka plugin is not included by default with fluentd and will need to be installed.

## Store Actions in Redis Store
If you wish to use Redis as an action store and to store the actions using fluentd you can use our fluentd plugin which can be installed with:

{% highlight bash %}
/usr/sbin/td-agent-gem install fluent-plugin-redis-store-seldon
{% endhighlight %}

The source github repo can be found [here](https://github.com/SeldonIO/fluent-plugin-redis-store). It is a fork of https://github.com/pokehanai/fluent-plugin-redis-store

You can then add to your central td-agent a match condifuration like:

{% highlight text %}
<match actions.**>
  type redis_store_seldon
  format_type json
  key_path userid
  value_path itemid
  key_prefix ActionHistory:
  key_prefix_path client
  key_prefix_sep :
  store_type zset
  value_expire 86400
</match>
{% endhighlight %}
