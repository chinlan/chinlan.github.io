---
layout: post
title: "Deploy notes"
description: "Deploy api backend app (Rails 6) and frontend app (Next.js + React.js + TypeScript) to AWS ECS"
categories: [devops]
tags: []
redirect_from:
  - /2020/07/26/
---
Problems occurs during the deploy process:

- Note the total `mem_limit` of the containers in the task definition should not exceed the EC2 instance memory size.
-> if memory is not enough, the tasks would keep getting stopped and restart (Exit: 137)

- Rails:
  - `config.hosts` setting
  - `rack-cors` , `config.allowed_cors_origins` (using AWS ALB, same origin so set to [])
  - Need to add asset precomiple step in the Dockerfile for deploy (Dockerfile-deploy)
  - Need to set environment variable: RAILS_MASTER_KEY for asset precompile.

- React.js + typescript + next.js
  - Even if `npm run dev` successed, `npm run build` could fail. Try building in local development environment before pushing.
  - See fixes: [link](http://chinlan.github.io/blog/2020/07/23/typescript-fixes-for-building-production-app/)
  - next.js: `Router.push(path)` not working
    [ref](https://github.com/vercel/next.js/issues/5264)
    Fix: import an empty `index.css` file in `_app.tsx` file.
    `styles/index.css`
    ~~~ css
    body {}
    ~~~
    `_app.tsx`
    ~~~tsx
    import '../styles/index.css'
    ~~~

- To disable host checking, set `config.hosts.clear` or `config.hosts = nil` in <environment>.yml
-> ALB host of healthcheck is hard to specifically assigned, return 403 error, causing target group become unhealthy

- This time not using 'service discovery', but got some knowledge:
  - To use `service discovery`, enable the `DNS hostnames` and `DNS resolution` of the VPC.
  - If container using `bridge` network type, `service discovery` will generate SRV record and an A record. But `<service name>.<private hosted zone name>` format which is meant to be used in the application code is the SRV record.
  - If you want to using A record, container network type will need to revised to `awsvpc` type, because I also use Application Load Balancer, so the target group will need to change from `instance` type to `ip` type. Don't need to create it manually, recreating the AWS ECS service and associate it to the Load Balancer again, the `ip` type target group can be created during this process.





