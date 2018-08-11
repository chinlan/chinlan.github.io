---
layout: post
title: "Rails: docker-compose add elasticsearch"
description: ""
categories: [docker rails elasticsearch]
tags: []
redirect_from:
  - /2018/08/11/
---

Gemfile:
~~~
gem 'searchkick'
~~~
docker-compose.yml
~~~~~~~~~~~~~~~~~~~~~
# ... other services...
elasticsearch:
  build:
    context: .
    dockerfile: docker/elasticsearch/Dockerfile
  environment:
    - xpack.security.enabled=false
  volumes:
    - es-data:/usr/share/elasticsearch/data
  ports:
    - 9200:9200
app:
  env_file:
    - .env
  build:
    context: .
    dockerfile: docker/app/Dockerfile
  volumes:
    - .:/app
    - bundle:/usr/local/bundle
  command: bundle exec rails s -p 3000 -b '0.0.0.0'
  ports:
    - "3000:3000"
  depends_on:
    - postgres
    - redis
    - elasticsearch
# ...volumes...
~~~~~~~~~~~~~~~~
docker/elasticsearch/Dockerfile
~~~~~~~~~~~~~~~~~
FROM docker.elastic.co/elasticsearch/elasticsearch:6.3.0

# plugins
RUN elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.0/elasticsearch-analysis-ik-6.3.0.zip
~~~~~~~~~~~~~~~~~~
initializers/elasticsearch.rb
~~~~~~~~~~~~~~~~~~
ENV['ELASTICSEARCH_URL'] = "http://elasticsearch:9200"
# default is 'http://localhost:9200'
~~~~~~~~~~~~~~~~~~
[ref](https://stackoverflow.com/questions/48744892/docker-rails-elastic-search)
