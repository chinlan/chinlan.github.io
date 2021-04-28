---
layout: post
title: "Using proxy to access IP limited external api endoints"
description: ""
categories: [rails]
tags: [docker proxy]
redirect_from:
  - /2021/03/01/
---

Our application needs to call an external API which has an IP address limit.

First solution:
  Using docker environment variables: `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`

Development:
docker/variables.env
~~~env
http_proxy=http://<user>:<password>@<proxy host IP>:<port>
https_proxy=http://<user>:<password>@<proxy host IP>:<port>
no_proxy=localhost,elasticsearch,lambda
~~~

Staging: (AWS ECS) (terraform)
Set environment variables in container_definitions:
/staging/ecs-web.tf
~~~tf
{
  "name": "http_proxy",
  "value": "${var.http_proxy}"
},
{
  "name": "https_proxy",
  "value": "${var.https_proxy}"
},
{
  "name": "no_proxy",
  "value": "${aws_elasticsearch_domain.tabi-two-staging-es74.endpoint},*.${var.domain}"
}

~~~

Note:
AWS elasticsearch VPC endpoint can only be accessed from inside VPC, so the elasticsearch VPC endpoint needs to be specified in no_proxy:
https://d0.awsstatic.com/aws-answers/Accessing_VPC_Endpoints_from_Remote_Networks.pdf


Second solution:
  In application code, specify to use proxy only when calling the IP address limited external API endpoints.

~~~ruby
include HTTParty

base_uri "#{EXTERNAL_API_HOST}/"
http_proxy ENV['PROXY_HOST'], ENV['PROXY_PORT'], ENV['PROXY_USER'], ENV['PROXY_PASSWORD']

def send_request
  # ......
  # ex. self.class.post(API_PATH, params.merge(opts))
end
~~~

References (Httparty source code):
https://github.com/jnunemaker/httparty/blob/f35528e240c4ff47df2b7bf4858845b4cb4b423d/lib/httparty.rb




