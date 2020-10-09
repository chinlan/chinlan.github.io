---
layout: post
title: "Send AWS Cloudwatch alarm through SNS notification to Slack"
description: ""
categories: [devops]
tags: [Terraform, AWS, DNS]
redirect_from:
  - /2019/09/20/
---

Step 1.
Create a slack app, activate its callback url. (Create a channel for receiving the notification message if not yet exists)

Step 2.
Create IAM role for executing lambda function.
Will need to attach 2 policies:
AmazonSSMFullAccess
AWSLambdaBasicExecuteRole

Step 3.
Need a KMS key to encrypt the slack callback url later. (Create 1 if not exists yet)
Permit the IAM role in step 2 to use this key by editing the key policy.
https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-example

Step 4.
Create a SNS topic with the role in step 2

Step 5.
Create a lambda function with the topic in step 4, and the KMS key in step 3, encrypt the slack settings, using blueprint python3 script.

Step 6.
Test lambda function:
- Create test event, revise the message format to cater to slack message format.
~~~
.......
  "Message": "{\"AlarmName\": \"Test Alarm\",\"NewStateValue\":\"xxxxxxx\",\"NewStateReason\":\"yyyyyyyyy\"}",
.......
~~~

Step 7.
Create Cloudwatch alarms:
- CPU Utility over 70% within 1 hour
- Memory Utility over 70% within 1 hour

[reference](https://medium.com/verybuy-dev/%E5%B0%87-aws-cloudwatch-alarms-%E7%99%BC%E4%BD%88%E5%88%B0-slack-c283959a90ca)
