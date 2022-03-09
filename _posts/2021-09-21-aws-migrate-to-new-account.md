---
layout: post
title: "AWS: migrate to new account"
description: ""
categories: [aws]
tags: []
redirect_from:
  - /2021/09/21/
---



- change account settings in terraform/circleCI env/local docker

- Migrate hosted zone
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-migrating.html
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-transfer-between-aws-accounts.html

- Copy s3 objects
https://aws.amazon.com/tw/premiumsupport/knowledge-center/account-transfer-s3/#:~:text=You%20can't%20transfer%20Amazon,objects%20to%20the%20destination%20account

- Migrate DynamoDB
https://aws.amazon.com/tw/premiumsupport/knowledge-center/dynamodb-cross-account-migration/

- Migrate Lambda
https://aws.amazon.com/tw/premiumsupport/knowledge-center/lambda-function-migration-console/

- Moving verified domains from SES
https://forums.aws.amazon.com/thread.jspa?threadID=118146
https://aws.amazon.com/tw/blogs/messaging-and-targeting/can-i-use-multiple-aws-accounts-with-ses/
https://aws.amazon.com/tw/premiumsupport/knowledge-center/ses-verify-domain-accounts-regions/

- Copy RDS snapshots
https://aws.amazon.com/tw/premiumsupport/knowledge-center/account-transfer-rds/
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html
For RDS, the snapshots of an unencrypted DB instance can be shared with a specific account, or the snapshots can be made public.


- terraform remote state
https://shinglyu.com/web/2019/09/30/moving-aws-service-across-accounts-using-terraform.html
