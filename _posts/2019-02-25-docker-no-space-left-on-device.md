---
layout: post
title: "Docker: no space left on device"
description: ""
categories: [docker]
tags: []
redirect_from:
  - /2019/02/25/
---

[ref](https://forums.docker.com/t/no-space-left-on-device-error/10894/15)

`docker rm $(docker ps -q -f 'status=exited')`
`docker rmi $(docker images -q -f "dangling=true")`

or

`docker images --filter ‘dangling=true’ | awk ‘{print $3}’ | grep -v IMAGE | xargs docker rmi`

[For more references](https://linuxize.com/post/how-to-remove-docker-images-containers-volumes-and-netw)

