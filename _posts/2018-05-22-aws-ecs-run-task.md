---
layout: post
title: "AWS: ecs run-task"
description: ""
categories: [AWS]
tags: []
redirect_from:
  - /2018/05/22/
---

We can use ecs run-task to execute tasks after deployment.

Prepare json file like this in local project directory,

app/overrides.json
~~~
{
  "containerOverrides": [
    {
      "name": "batch",
      "command": ["bundle", "exec", "rake", "db:migrate"]
    }
  ]
}
~~~

Then run in local terminal:
~~~
aws ecs run-task \

  --region ap-northeast-1 \

  --cluster xxx(cluster-name) \

  --task-definition yyy(task-definition-name) \

  --overrides file://overrides.json(path/to/json)
~~~

If this task is an one-off task, don't need to commit overrides.json, remove it after execution.

Safer:
Setting dry_run option in the task,
at first time execute with dry_run: true,
make sure it prints correct output in the log,
then execute with dry_run: false.
