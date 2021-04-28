---
layout: post
title: "Nginx as reverse proxy in docker-compose development environment"
description: ""
categories: [devops]
tags: [docker proxy]
redirect_from:
  - /2020/11/23/
---
References:
https://www.domysee.com/blogposts/reverse-proxy-nginx-docker-compose
http://nginx.org/en/docs/http/ngx_http_core_module.html#location
https://linuxize.com/post/nginx-log-files/
https://stackoverflow.com/questions/46153773/rails-passenger-nginx-error-request-origin-domainname-com-does-not-match-r

docker-compose.yml
~~~yml
version: "2.3"

services:
  db:
    hostname: db
    restart: unless-stopped
    build:
      context: .
      dockerfile: ./Dockerfile-db
    ports:
      - 3306:3306
    volumes:
      - /var/lib/mysql
    mem_limit: 512M
    cpus: 0.5
    cap_add:
      - SYS_NICE # CAP_SYS_NICE

  redis:
    restart: unless-stopped
    image: redis:alpine
    mem_limit: 128M
    cpus: 0.2

  elasticsearch:
    restart: unless-stopped
    build: ./docker/elasticsearch
    environment:
      - discovery.type=single-node
      - discovery.zen.minimum_master_nodes=1
      - xpack.security.enabled=false
      - network.host=0.0.0.0
      - http.port=9200
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ports:
      - 9200:9200
      - 9300:9300
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1536M
    mem_reservation: 1G
    oom_kill_disable: true
    cpus: 1
    healthcheck:
      test: ["CMD-SHELL", 'curl -XGET "http://localhost:9200/_cluster/health" | jq -r ".status" | grep -q -e "yellow" -e "green"']
      interval: 2s
      timeout: 5s
      retries: 50
      start_period: 60s

  mongo:
    restart: unless-stopped
    image: mongo:4.2
    mem_limit: 1G
    cpus: 0.8
  frontend:
    restart: unless-stopped
    working_dir: /frontend
    # ports:
    #   - 4000:4000
    expose:
      - '4000'
    build:
      context: .
      dockerfile: ./Dockerfile-frontend
    mem_limit: 1G
    memswap_limit: 1G
    mem_reservation: 512M
    oom_kill_disable: true
    cpus: 1
    env_file:
      - frontend_variables.env
  web:
    restart: unless-stopped
    build: .
    working_dir: /app
    ports:
      - 3000:3000 # Access assets and images directly from 3000(the proxy setting is only for local development)
    # expose:
    #   - '3000'
    logging:
      driver: "json-file"
      options:
        max-size: "1k"
        max-file: "3"
    depends_on:
      db:
        condition: service_started
      redis:
        condition: service_started
      elasticsearch:
        condition: service_healthy
      mongo:
        condition: service_started
    env_file:
      - variables.env
    cpus: 2
    mem_limit: 2G
    mem_reservation: 512M
    oom_kill_disable: true

  reverse-proxy:
    image: nginx:latest
    restart: unless-stopped
    ports:
      - '8000:80'
      # - '4443:443'
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/error.log:/etc/nginx/error_log.log
      - ./nginx/cache/:/etc/nginx/cache
      # - /etc/letsencrypt/:/etc/letsencrypt/
    depends_on:
      - frontend
      - web

networks:
  default:
    external:
      name: job-network
~~~

nginx.conf
~~~
events {
  worker_connections 1024; # Default
}

http {
  error_log /etc/nginx/error_log.log warn;
  proxy_cache_path /etc/nginx/cache keys_zone=one:10m;

  server {
    listen 80;
    server_name localhost;
    proxy_cache one;

    location = / {
      proxy_pass http://frontend:4000;
    }

    location /api {
      proxy_pass http://web:3000/api;
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr:8000;
    }

    location = /healthcheck {
      proxy_pass http://web:3000/healthcheck;
    }

    location = /sitemap {
      proxy_pass http://web:3000/sitemap;
    }

    location = /admins {
      proxy_pass http://web:3000/admins;
    }

    location /admins {
      proxy_pass http://web:3000/admins;
      proxy_set_header Host $http_host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # For CSRF token
    }

    location /admin_users {
      proxy_pass http://web:3000/admin_users;
      proxy_set_header Host $http_host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Real-IP $remote_addr:8000;
      proxy_set_header X-Forwarded-Port $server_port;
    }

    location /users {
      proxy_pass http://web:3000/users;
    }

    location /rails {
      proxy_pass http://web:3000/rails;
    }

    location /assets {
      proxy_pass http://web:3000/assets;
    }

    location /images {
      proxy_pass http://web:3000/images;
    }

    location / {
      proxy_pass http://frontend:4000;
    }
  }
}
~~~

From inside the frontend container, cannot use `localhost`, need to specify the docker service name.
frontend_variables.env (for Axios)
~~~
BASE_URL=http://localhost:8000
API_BASE_URL=http://reverse-proxy:80
~~~

If using `fetch`, just using `relative path` will work. EX. `/api/xxx`.

Remove all the cors settings:
~~~js
// "Access-Control-Allow-Origin": "*",
...
// mode: "cors",
~~~

Remove cors setting and add host setting in Rails side (Backend):
development.rb
~~~rb
  config.hosts << 'web'
  config.hosts << 'frontend'
  config.hosts << 'localhost:8000'
  config.allowed_cors_origins = []
~~~

If assets and images also need to be served by Nginx reverse proxy:
config/initializers/carrierwave.rb
~~~
config.asset_host = "http://localhost:8000"
~~~
development.rb
~~~
config.action_controller.asset_host = 'http://localhost:8000'
config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect'
~~~

