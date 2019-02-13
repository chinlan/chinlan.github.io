---
layout: post
title: "Elasticsearch: high disk watermark exceeded on one or more nodes"
description: ""
categories: [elasticsearch]
tags: [docker]
redirect_from:
  - /2019/01/23/
---
~~~
[2019-01-23T01:12:46,735][INFO ][o.e.c.r.a.DiskThresholdMonitor] [GqPVEhd] rerouting shards: [high disk watermark exceeded on one or more nodes]

elasticsearch6_1  | [2019-01-23T01:12:57,686][WARN ][o.e.l.LicenseService     ] [GqPVEhd]

elasticsearch6_1  | #

elasticsearch6_1  | # LICENSE [EXPIRED] ON [WEDNESDAY, JULY 11, 2018]. IF YOU HAVE A NEW LICENSE, PLEASE UPDATE IT.

elasticsearch6_1  | # OTHERWISE, PLEASE REACH OUT TO YOUR SUPPORT CONTACT.

elasticsearch6_1  | #

elasticsearch6_1  | # COMMERCIAL PLUGINS OPERATING WITH REDUCED FUNCTIONALITY

elasticsearch6_1  | # - security

elasticsearch6_1  | #  - Cluster health, cluster stats and indices stats operations are blocked

elasticsearch6_1  | #  - All data operations (read and write) continue to work

elasticsearch6_1  | # - watcher

elasticsearch6_1  | #  - PUT / GET watch APIs are disabled, DELETE watch API continues to work

elasticsearch6_1  | #  - Watches execute and write to the history

elasticsearch6_1  | #  - The actions of the watches don't execute

elasticsearch6_1  | # - monitoring

elasticsearch6_1  | #  - The agent will stop collecting cluster and indices metrics

elasticsearch6_1  | #  - The agent will stop automatically cleaning indices older than [xpack.monitoring.history.duration]

elasticsearch6_1  | # - graph

elasticsearch6_1  | #  - Graph explore APIs are disabled

elasticsearch6_1  | # - ml

elasticsearch6_1  | #  - Machine learning APIs are disabled

elasticsearch6_1  | # - logstash

elasticsearch6_1  | #  - Logstash specific APIs are disabled. You can continue to manage and poll stored configurations

elasticsearch6_1  | # - deprecation

elasticsearch6_1  | #  - Deprecation APIs are disabled

elasticsearch6_1  | # - upgrade

elasticsearch6_1  | #  - Upgrade API is disabled

elasticsearch6_1  | [2019-01-23T01:13:16,708][WARN ][o.e.c.r.a.DiskThresholdMonitor] [GqPVEhd] high disk watermark [90%] exceeded on [GqPVEhdDRe-mvv0U-A-QZw][GqPVEhd][/usr/share/elasticsearch/data/nodes/0] free: 3.6gb[5.8%], shards will be relocated away from this node

elasticsearch6_1  | [2019-01-23T01:13:46,850][WARN ][o.e.c.r.a.DiskThresholdMonitor] [GqPVEhd] high disk watermark [90%] exceeded on [GqPVEhdDRe-mvv0U-A-QZw][GqPVEhd][/usr/share/elasticsearch/data/nodes/0] free: 3.6gb[5.8%], shards will be relocated away from this node

elasticsearch6_1  | [2019-01-23T01:13:46,851][INFO ][o.e.c.r.a.DiskThresholdMonitor] [GqPVEhd] rerouting shards: [high disk watermark exceeded on one or more nodes]

~~~
The above is the error log I got recently telling me that something went wrong.

And indices became read-only though I didn't do any revise in the setting.

After some research,
[ref](https://benjaminknofe.com/blog/2017/12/23/forbidden-12-index-read-only-allow-delete-api-read-only-elasticsearch-indices/)

I ran this in the elasticsearch6 container bash. (We use docker)
~~~
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/_all/_settings -d
~~~
the response:
~~~
'{"index.blocks.read_only_allow_delete": null}'
{"acknowledged":true}
~~~~

[ref](https://github.com/elastic/elasticsearch/issues/16082)

Also ran this:
~~~
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/_cluster/settings -d '{"persistent": {"cluster.routing.allocation.disk.threshold_enabled":false}}'
~~~
the response:
~~~
{"acknowledged":true,"persistent":{"cluster":{"routing":{"allocation":{"disk":{"threshold_enabled":"false"}}}}},"transient":{}}
~~~
Now the indices are no longer read-only, new record inserted and old record updated will make a write to them.

Wait and observe if anything is still wrong.

