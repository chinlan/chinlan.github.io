---
layout: post
title: "AWS: Copy production database to test migation speed"
description: ""
categories: [AWS]
tags: []
redirect_from:
  - /2018/08/07/
---

RDS snapshots: find the latest production db snapshot,

Select restore, x4-4large(the same as production setting), choose development network to not have any effect on production.

Wait for it to be created.

When it is created, change database.yml dev host to the created test db endpoint.

Comment out migration part from bin/deploy script.

Deploy to dev from local: `bin/deploy dev`

After deployment, ssh to dev, enter container, do `time rails db:migrate`, see how much time it takes, done.


Remember to delete the test db instance.

And revise back all the code changed above.
